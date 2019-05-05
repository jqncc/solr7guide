## 从Ruby使用Solr 
<div class="content-intro view-box ">
## 从Ruby使用Solr

Solr有一个可选的Ruby响应格式，它扩展了JSON输出以允许Ruby的解释器安全地评估响应。
      
  
这种Ruby响应格式与JSON的不同之处在于：  

    - Ruby的单引号字符串用于防止可能的字符串利用：<ul><li>\ 和 ' 是仅有的两个转义字符；
- unicode转义不使用,数据被写为原始的UTF-8
</li>
    <li>nil 用于 null</li>
    <li>=&gt; 用作映射中的键/值（key/value）分隔符</li>
</ul>
下面是一个来自Solr的Ruby响应示例，其中包含一个请求：http://localhost:8983/solr/techproducts/select?q=iPod&amp;wt=ruby&amp;indent=on（使用Solr启动bin/solr start -e techproducts）：  
```
{
  'responseHeader'=&gt;{
    'status'=&gt;0,
    'QTime'=&gt;0,
    'params'=&gt;{
      'q'=&gt;'iPod',
      'indent'=&gt;'on',
      'wt'=&gt;'ruby'}},
  'response'=&gt;{'numFound'=&gt;3,'start'=&gt;0,'docs'=&gt;[
      {
        'id'=&gt;'IW-02',
        'name'=&gt;'iPod &amp; iPod Mini USB 2.0 Cable',
        'manu'=&gt;'Belkin',
        'manu_id_s'=&gt;'belkin',
        'cat'=&gt;['electronics',
          'connector'],
        'features'=&gt;['car power adapter for iPod, white'],
        'weight'=&gt;2.0,
        'price'=&gt;11.5,
        'price_c'=&gt;'11.50,USD',
        'popularity'=&gt;1,
        'inStock'=&gt;false,
        'store'=&gt;'37.7752,-122.4232',
        'manufacturedate_dt'=&gt;'2006-02-14T23:55:59Z',
        '_version_'=&gt;1491038048794705920},
      {
        'id'=&gt;'F8V7067-APL-KIT',
        'name'=&gt;'Belkin Mobile Power Cord for iPod w/ Dock',
        'manu'=&gt;'Belkin',
        'manu_id_s'=&gt;'belkin',
        'cat'=&gt;['electronics',
          'connector'],
        'features'=&gt;['car power adapter, white'],
        'weight'=&gt;4.0,
        'price'=&gt;19.95,
        'price_c'=&gt;'19.95,USD',
        'popularity'=&gt;1,
        'inStock'=&gt;false,
        'store'=&gt;'45.18014,-93.87741',
        'manufacturedate_dt'=&gt;'2005-08-01T16:30:25Z',
        '_version_'=&gt;1491038048792608768},
      {
        'id'=&gt;'MA147LL/A',
        'name'=&gt;'Apple 60 GB iPod with Video Playback Black',
        'manu'=&gt;'Apple Computer Inc.',
        'manu_id_s'=&gt;'apple',
        'cat'=&gt;['electronics',
          'music'],
        'features'=&gt;['iTunes, Podcasts, Audiobooks',
          'Stores up to 15,000 songs, 25,000 photos, or 150 hours of video',
          '2.5-inch, 320x240 color TFT LCD display with LED backlight',
          'Up to 20 hours of battery life',
          'Plays AAC, MP3, WAV, AIFF, Audible, Apple Lossless, H.264 video',
          'Notes, Calendar, Phone book, Hold button, Date display, Photo wallet, Built-in games, JPEG photo playback, Upgradeable firmware, USB 2.0 compatibility, Playback speed control, Rechargeable capability, Battery level indication'],
        'includes'=&gt;'earbud headphones, USB cable',
        'weight'=&gt;5.5,
        'price'=&gt;399.0,
        'price_c'=&gt;'399.00,USD',
        'popularity'=&gt;10,
        'inStock'=&gt;true,
        'store'=&gt;'37.7752,-100.0232',
        'manufacturedate_dt'=&gt;'2005-10-12T08:00:00Z',
        '_version_'=&gt;1491038048799948800}]
  }}
```
下面是一个简单的例子，说明如何使用Ruby响应格式来查询Solr：  
```
require 'net/http'
h = Net::HTTP.new('localhost', 8983)
http_response = h.get('/solr/techproducts/select?q=iPod&amp;wt=ruby')
rsp = eval(http_response.body)
puts 'number of matches = ' + rsp['response']['numFound'].to_s
#print out the name field for each returned document
rsp['response']['docs'].each { |doc| puts 'name field = ' + doc['name'] }
```
对于与Solr的简单交互，这可能是所有你需要的！如果您正在使用Solr建立复杂的交互，请参考：https://wiki.apache.org/solr/Ruby%20Response%20Format  
