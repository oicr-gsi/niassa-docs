{% include functions.liquid %}

\> Back to [{{ page.www-bit}}]({{version_url}}/web-service/{{page.www-bit}})

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