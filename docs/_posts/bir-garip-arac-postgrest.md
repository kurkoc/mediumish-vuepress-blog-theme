---
tags:
- postgresql
- " rest"
- api
title: 'bir garip araç : postgrest'
date: 2020-10-18 21:00:00 +0000
author: kurkoc
featuredimg: ''
summary: ''

---
frontend tarafında bir şeyleri kurcalarken bir an geliyor ve uzak sunucu üzerinde çalışan gerçeğe yakın verilerle çalışma ihtiyacımız oluyor. bu ihtiyacı karşılamak üzere internette yüzlerce hazır rest servis hizmeti bulunmakta. bunlardan bazıları sadece select işlemleri üzerine kurgulanmışken bazıları da bütün crud işlemlerine olanak sağlamakta. isterseniz kolaylıkla evcil hayvan, makale, çalışan sistemi geliştirirken bulabilirsiniz kendinizi.  çok gelişmiş olanlarda veri modelinizi dinamik oluşturup rastgele veriler vs. oluşturabiliyorsunuz. bunları zaten biliyorsunuzdur uzatmaya gerek yok.

bu servisler genel olarak aradığımızı bize sunmakla birlikte bazen daha farklı bir yapı isteyebilir ya da kendi kurguladığınız bir veritabanı modeli üzerinde çalışmak isteyebilirsiniz. 

bahsedeceğim araç, postgresql veritabanı üzerinde çalışıp, veritabanı tablolarını tam teşekküllü bir rest servisine çeviren [postgrest](http://postgrest.org/). kısaca bahsetmek gerekirse; aşina olduğunuz http verb'lerini kullanarak basit http çağrıları yardımıyla veritabanı üzerinde crud işlemleri yapabilmemize olanak sağlayan bir araç diyebiliriz.

contacts tablomuz üzerinde basit bir select sorgusu çalıştıralım.

    GET /contacts HTTP/1.1

tablomuza yeni bir kayıt ekleyelim.

    POST /contacts HTTP/1.1
    Content-Type: application/json
    { "firstname": "jane", "lastname": "doe" }

yaşı 25'ten büyük kişileri listeleyelim.

    GET /contacts?age=gt.25 HTTP/1.1

yaşı 25'ten büyük kişileri sadece isim ve yaşları görülecek şekilde listeleyelim.

    GET /contacts?age=gt.25&select=first_name,age HTTP/1.1

sıralama yapalım.

    GET /contacts?order=age.desc,height.asc HTTP/1.1

sayfalama yapalım

    GET /contacts?limit=15&offset=30 HTTP/1.1