[nProbe v7](www.ntop.org/products/nprobe/) integration with [Facetflow](https://facetflow.com/)
================



## ![](http://www.ntop.org/wp-content/uploads/2011/08/nboxLogo.gif) Setup:

In this setup guide, we'll be using:

* nprobe: capturing packets and sending json reports to Facetflow's ES Bulk API *(with basic auth)*
  * nprobe: "Elasticsearch JSON Export" plugin with direct Bulk insert and indexing functionality **
* qbana: connecting to facetflow w/ user authentication option *(hosted locally or remotely)*

*Note: A free or paid account at [Facetflow](https://facetflow.com/) is required.*
**Note: Requires a PRO license from [ntop.org](http://www.nmon.net/shop/)
<br>

## ![](http://www.ntop.org/wp-content/uploads/2011/08/nboxLogo.gif) Installation
### nProbe Setup

To install nProbe on your OS:

* Ubuntu 12.04 LTS
```
wget http://www.nmon.net/apt/12.04/all/apt-ntop.deb
sudo dpkg -i apt-ntop.deb
apt-get clean all
apt-get update
apt-get install pfring nprobe
```
* Ubuntu 14.04 LTS
```
wget http://www.nmon.net/apt/14.04/all/apt-ntop.deb
sudo dpkg -i apt-ntop.deb
apt-get clean all
apt-get update
apt-get install pfring nprobe
```
* Debian 7.6 (stable)
```
wget http://www.nmon.net/apt/7.6/all/apt-ntop.deb
sudo dpkg -i apt-ntop.deb
apt-get clean all
apt-get update
apt-get install pfring nprobe
```

*Additional OS download are available at: [http://www.nmon.net/packages/](http://www.nmon.net/packages/)*


### FacetFlow Setup
Sign up for a free or paid account/package at [FacetFlow.com](http://www.FacetFlow.com) and obtain your *{API_KEY}* and *{HOST_ID}*
```
The {API_KEY} will be used to authenticate both QBana users and other Clients reading/inserting data.
```

### Qbana/Kibana Setup

```
cd /usr/src
git clone https://github.com/QXIP/Qbana
mkdir /usr/share/nginx/qbana
cp -r Qbana/src/* /usr/share/nginx/qbana/
```
Edit the elasticsearch parameter in /usr/share/nginx/qbana/config.js:
```
 elasticsearch: { server: "https://{YOUR_HOST_ID}.facetflow.io", withCredentials: true }
```



----------------

<br>

## ![](http://www.ntop.org/wp-content/uploads/2011/08/nboxLogo.gif) Configuration



----------------

<br>

### nProbe *(dummy template, adjust according to requirements)*
```
$ nprobe -T "%IPV4_SRC_ADDR %L4_SRC_PORT %IPV4_DST_ADDR %L4_DST_PORT %PROTOCOL %IN_BYTES %OUT_BYTES %FIRST_SWITCHED %LAST_SWITCHED %IN_PKTS %OUT_PKTS %IP_PROTOCOL_VERSION %APPLICATION_ID %L7_PROTO_NAME %ICMP_TYPE %SRC_IP_COUNTRY %DST_IP_COUNTRY %APPL_LATENCY_MS" --redis localhost --elastic "nProbe;nprobe;http://127.0.0.1:9200/_bulk" -b 1 -i any --json-labels -t 30
```

----------------

## ![](http://www.ntop.org/wp-content/uploads/2011/08/nboxLogo.gif) You're Done! 

nProbe template metrics will now appear in QBana

![](http://i.imgur.com/9gXTKCd.png)

Qbana comes preloaded with nProbe optimized dashboards. If you prefer using vanilla Kibana you absolutely can, just import our nProbe Template dashboards as starters or model your own. 

For more information about nProbe visit: http://www.ntop.org/products/nprobe/


<br>

-------------------------

### Quick Tips

* Send a test entry locally:
```
$ curl -u {YOUR_API_KEY}: \
       -XPOST 'http://127.0.0.1:19200/my_index/posts' -d '{
    "user": "myself",
    "post_date": "2014-09-24T09:48:41.328Z",
    "message": "trying out facetflow and nprobe"
  }'
```

* Drop your indexes in sandbox mode:
```
curl -XDELETE 'http://127.0.0.1:19200/my_index/' -u {API_KEY}
```

* Count & Rotate nProbe indexes in your limited sandbox from CRON (scripts/)
```
curl -XGET "http://localhost:19200/nprobe-*/_count" -u {YOUR_API_KEY}: -d '{"query": {"match_all": {} } }' -s
```

```
curl -XDELETE "http://localhost:19200/nprobe-*/" -u {YOUR_API_KEY}:
```

* If you're parsing syslog lines too, make sure your hosts are time/ntp sycronized

* When using several nProbe instances/plugins, create new in/out pairs with custom "_type" in nProbe

* In Facetflow, generate custom keys for your agents/applications and use the ACL settings to improve security

* Use NGINX to Proxy Requests to Facetflow *(optional)*

Nginix can be used to receive and forward local http traffic to your Facetflow instance using https and passing along the basic authentication.

Paste and Edit the following configuration in /etc/nginx/sites-enabled/facetflow_fwd.conf
```
server {
    listen       19200;
    server_name  {YOUR_HOST_ID}.facetflow.io;lo

    error_log   facetflow-errors.log;
    access_log  facetflow.log;

    location / {

      # Deny access to Cluster API
      if ($request_filename ~ "_cluster") {
        return 403;
        break;
      }
      # Pass requests to ElasticSearch
      proxy_pass https://{YOUR_HOST_ID}.facetflow.io;
      proxy_redirect off;
      proxy_set_header  Host $proxy_host;
      # Route all requests to authorized user's own index
      #rewrite  ^(.*)$ $1  break;
      rewrite_log on;
    }
}

```

------------
<br>

This guide is sponsored by: 
<br><br><center>
<a href="http://qxip.net" target="_blank"><img src="http://www.sipcapture.org/data/images/qxip.png"></a> <a href="http://ntop.org" target="_blank"><img src="http://www.ntop.org/wp-content/uploads/2011/08/logo_new_m.png"></a> <a href="http://facetflow.com" target="_blank"><img src="http://i.imgur.com/cIvYisr.png"></a>
</center>
