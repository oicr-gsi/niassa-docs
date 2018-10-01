{% include functions.liquid %}

{{ include.summary }}

## Resource URL

{{ include.method }} {{ include.server_name }} {{ include.uri }}

## Parameters

{% if include.show %}* show
: retrieve other objects linked to this one, e.g. `{{ uri }}?show=(table)`
{% endif %}
{% if include.id %}* id
: SeqWare accession for an object
{% endif %}
