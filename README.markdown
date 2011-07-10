# What is it ?

It's a ruby library to help tcpdump file processing : do some offline analysis on tcpdump files.

Main functionnalities :

* Rebuild tcp streams
* Extract and parse http request

# How use it

## Make a tcpdump

* `tcpdump -w out.pcap -s 4096 <filter>`
* Get the output file out.pcap

Please adjust the 4096 value, to the max packet size to capture.

## Write a ruby script

    require 'pcap_tools'

    # Load tcpdump file
    capture = Pcap::Capture.open_offline('out.pcap')

## Available functions

### Extract tcp streams

This function rebuild tcp streams from a pcap capture object.

    tcp_streams = PcapTools::extract_tcp_streams(capture)

`tcp_streams` is an array of hash, each hash has tree keys :

* `:type` : `:in` or `:out`, if the packet was sent or received
* `:time` : timestamp of packet
* `:data` : payload of packet

Remarks :

* Packets are in the rigth ordere
* Packets are not merged (eg an http response can be splitted on serval consecutive packets,
with the same type `:in` or `:out`). 
To reassemble packet of the same type, please use `stream.rebuild_packets`

### Extract http calls

This function extract http calls from an http stream, returned from the `extract_tcp_streams` function.

    http_calls = PcapTools::extract_http_calls(stream)

`http_calls` is an array of `http_call`.

An `http_call` is an array of two object

* The http request, an instance of `Net::HTTPRequest`, eg `Net::HTTPGet` or `Net::HTTPPost`. You can use this object
like any http request of [std lib `net/http`](http://www.ruby-doc.org/stdlib/libdoc/net/http/rdoc/index.html)
  * `req.path` : get the uri
  * `req['User-Agent']` : get the User-Agent
  * `req.body` : get the request body
  * ...
* The http response, an instance of `Net::HTTPResponse`, eg `Net::HTTPOk` or `Net::HTTPMovedPermanently  You can use this object
  like any http response of [std lib `net/http`](http://www.ruby-doc.org/stdlib/libdoc/net/http/rdoc/index.html)
  * `resp.code` : get the return code
  * `resp['User-Agent']` : get the User-Agent
  * `resp.body` : get the request body
  * ...

The response can be `nil` if there is no response in the tcp stream.

The request and response object have some new attributes

* `req.time` : get the time where the request or response was captured

For the response object, the following "Content-Encoding" type are honored :

* gzip

### Extract http calls from capture

The two in one : extract http calls from the capture object

    http_calls = PcapTools::extract_http_calls_from_capture(capture)