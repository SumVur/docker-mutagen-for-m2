FROM elasticsearch:7.9.3

ENV ES_PLUGIN_CMD=/usr/share/elasticsearch/bin/elasticsearch-plugin

RUN \
  $ES_PLUGIN_CMD install analysis-phonetic --verbose && \
  $ES_PLUGIN_CMD install analysis-icu

EXPOSE 9200 9300
