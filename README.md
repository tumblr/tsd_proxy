# tsd_proxy

A Clojure program that accepts incoming OpenTSDB requests (clients
dumping data points into OpenTSDB) and sends them on to multiple
backend OpenTSDB servers.  Since this is a one-way proxy, it doesn't
respond to status requests.

## Notes

The proxy looks for the config file at /etc/tsd_proxy.conf, it should
have the format below.  All of the four keys are required.

    {
      :end-points  ["localhost:7777" "localhost:7778"],
      :listen-port 9999,
      :junk-filter ["^put" "host=(?!localhost)"],
      :limit 10
    }

If the proxy cannot forward messages to the downstream consumers, it
buffers (upto :limit * incoming_message_rate) them if the underlying
jvm heap size can support it.  The :limit parameter gives you a knob
to help keep the proxy from getting into the Full-GC death loop.

The proxy will continue trying to reach the consumers every few
seconds.  Once the consumer is up, it sends the buffered input to the
consumer in the order it was received.  If you can allocate a lot of
memory to the jvm, you should set the :limit to be reasonably high.

You can enable filters on the incoming data stream by setting the
:junk-filter array to a regex pattern list.  All of these patterns
should match for a message to be sent to the downstream consumers.

Specifically, the regex patterns you supply are converted to a java
regex pattern and used with java.util.regex.Matcher.find to determine
if there is a match.  If you turn this feature off (set :junk-filter
to an empty array [] in the config) - you will see better performance
(CPU wise).

## Usage

lein run (or lein trampoline run) or make an uberjar (lein uberjar)
and run it with "java -jar UBERJAR.jar"

## License

Copyright © 2013 Aravind Gottipati

Distributed under the Apache License, Version 2.0.
http://www.apache.org/licenses/LICENSE-2.0
