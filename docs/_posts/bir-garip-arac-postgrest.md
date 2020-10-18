---
tags:
- postgresql
- " rest"
- api
title: 'kurcalama : postgrest'
date: 2020-10-18 21:00:00 +0000
author: kurkoc
featuredimg: "/assets/img/logo-1.png"
summary: ''

---
bu aralar biraz vuejs inceliyorum. bu aralar ve genel olarak frontend tarafında bir şeyleri kurcalarken bir an geliyor ve uzak sunucu üzerinde çalışan, gerçeğe yakın verilerle çalışma ihtiyacınız oluyor. bu ihtiyacı karşılamak üzere internette yüzlerce hazır rest servis hizmeti bulunmakta. bunlardan bazıları sadece select işlemleri üzerine kurgulanmışken bazıları da bütün crud işlemlerine olanak sağlamakta. isterseniz kolaylıkla evcil hayvan dükkanı, makale sistemi, çalışan uygulaması geliştirirken bulabilirsiniz kendinizi.  çok gelişmiş olanlarda veri modelinizi dinamik oluşturup rastgele veriler vs. oluşturabiliyorsunuz. bunları zaten biliyorsunuzdur uzatmaya gerek yok.

bu servisler genel olarak aradığımızı bize sunmakla birlikte bazen daha farklı bir yapı isteyebilir ya da kendi kurguladığınız bir veritabanı modeli üzerinde çalışmak isteyebilirsiniz.

![](/assets/img/logo-1.png)

bahsedeceğim araç, postgresql veritabanı üzerinde çalışıp, veritabanı tablolarını tam teşekküllü bir rest servisine çeviren [postgrest](http://postgrest.org/). kısaca bahsetmek gerekirse; aşina olduğunuz http verb'lerini kullanarak basit http çağrıları yardımıyla veritabanı üzerinde crud işlemleri yapabilmemize olanak sağlayan bir araç diyebiliriz.

örnek olarak; contacts adında bir tablomuz olsun. uygulamayı çalıştırıp aşağıdaki gibi GET request'i attığımızda contacts tablomuzun içeriği json olarak listeleniyor desem.

    GET /contacts HTTP/1.1

bence güzel geliyor kulağa.

benim SampleDatabase adında bir database'im var ve içerisinde örnek olması açısından oluşturulmuş "category" ve "product" adında iki adet tablo var.

![](/assets/img/db_schema.png)

uygulamanın indirme ve kurulum kısımlarını es geçiyorum. burada dikkat etmeniz gereken postgrest uygulaması için oluşturulan kullanıcının select, insert, update vs. yapabilmesi için ilgili veritabanı üzerindeki privilege'larının verilmiş olması gerekiyor.

![](/assets/img/run.PNG)

uygulama çalıştırıldığında default olarak 3000 portu üzerinden bir rest servis ayağa kaldırıyor aslında.

ilk request olarak product tablomuzun içeriğini listeleyelim;

    GET /product HTTP/1.1
    
    [
      {
        "id": 1,
        "name": "asus",
        "category_id": 1,
        "quantity": 10
      },
      {
        "id": 2,
        "name": "fujitsu",
        "category_id": 1,
        "quantity": 6
      },
      {
        "id": 3,
        "name": "hp",
        "category_id": 1,
        "quantity": 33
      },
      {
        "id": 4,
        "name": "iphone",
        "category_id": 2,
        "quantity": 3
      },
      {
        "id": 5,
        "name": "note",
        "category_id": 2,
        "quantity": 8
      },
      {
        "id": 6,
        "name": "xiamoi mi",
        "category_id": 2,
        "quantity": 12
      }
    ]

peki, tekil bir ürünü getirmek istersek. rest servislerin best practicelerini düşünürsek product/2 şeklinde bir query ile ya da hiç olmadı product?id=2 şeklinde bir query ile olması gerekir mantıken ancak burada graphql tarzı bir gelişmiş sorgulama yapısı sağlayabilmek için farklı bir yaklaşım geliştirilmiş.

    GET /product?id=eq.3 HTTP/1.1
    
    [
      {
        "id": 3,
        "name": "hp",
        "category_id": 1,
        "quantity": 33
      }
    ]

\** burada dikkat edilirse unique bir kolon id'ye göre bir filtreleme yaptık, sonuç olarak tek bir obje beklememize rağmen içerisinde tek bir eleman olan bir array döndü yine de bize. bu durum client kodumuzda hataya sebebiyet verebilir. bunu önlemek için Accept header'ı olarak application/vnd.pgrst.object+json  gönderebiliriz.

category_id'si 1 olan ve adeti 20 ve daha fazlası olan ürünler

    /product?category_id=eq.1&quantity=gte.20 HTTP/1.1

category_id kolonunun görülmesini istemiyorum ve sadece id, name, quantity fieldlarını görmek istiyoruz diyelim. (buradan sonraki örneklerde sonuç kümesini kısaltıyorum gereksiz uzamaması için)

    GET /product?select=id,name,quantity HTTP/1.1
    
    {
        "id": 1,
        "name": "asus",
        "quantity": 10
      }

stoğumuzda 10 ve üzerinde adet olan ürünleri listelemek istiyoruz.

    GET /product?select=id,name,quantity&quantity=gte.10 HTTP/1.1
    
     [
      {
        "id": 1,
        "name": "asus",
        "quantity": 10
      },
      {
        "id": 3,
        "name": "hp",
        "quantity": 33
      },
      {
        "id": 6,
        "name": "xiamoi mi",
        "quantity": 12
      }
    ]

adet sayısına göre sıralama yapalım.

    GET /product?select=id,name,quantity&quantity=gte.10&order=quantity.desc HTTP/1.1
    
    [
      {
        "id": 3,
        "name": "hp",
        "quantity": 33
      },
      {
        "id": 6,
        "name": "xiamoi mi",
        "quantity": 12
      },
      {
        "id": 1,
        "name": "asus",
        "quantity": 10
      }
    ]

sayfalama yapalım

    GET /product?limit=15&offset=30 HTTP/1.1

nosql tarzı bir embedded reference ilişkisi kuralım. ürünleri listelerken kategorisini de gidip foreign key ilişkisi üzerinden alalım.

    GET /product?select=id,name,quantity,category(id,name) HTTP/1.1
    
    [
      {
        "id": 1,
        "name": "asus",
        "quantity": 10,
        "category": {
          "id": 1,
          "name": "bilgisayar"
        }
      },
      ...
     ]

admin hep get requestleri attın, yok mu biraz post, put, delete diyorsunuz muhtemelen.

    POST /category HTTP/1.1
    Content-Type: application/json
    
    
    {
     "name" : "Klavye"
    }

Yeni bir kategori ekledik ve 201 status code'umuzu aldık karşılığında.

id'si 3 olan ürünümüzün adetini 44 yapalım.

    PUT /product?id=eq.3 HTTP/1.1
    Content-Type: application/json
    {
     	"id" : 3,
     	"name" : "hp",
     	"category_id" : 1,
     	"quantity" : 44
    }

Az önce adetini arttırdığımız ürünü silelim.

    DELETE /product?id=eq.3 HTTP/1.1

Evet ben bu araca bayıldım ancak bu örnekleri bir yerde sonlandırmam gerekiyor. Kullanılabilen [operatörler](http://postgrest.org/en/v7.0.0/api.html#operators) ve join işlemleri incelendiğinde bir rest servisten beklediğiniz filtreleme, sıralama, sayfalama, crud işlemler vs. hemen hemen her operasyonu yapabildiğinizi rahatlıkla söyleyebilirim ancak bazen karmaşık joinler, filtrelemeler ya da bazı sql yeteneklerini(case when gibi) kullanmanız gereken durumlar olabilir. Bu tarz senaryolarda da view, stored procedure tanımlayıp benzer şekilde çağırabiliyoruz.

Söyleyeceklerim bu kadar, iyi günler