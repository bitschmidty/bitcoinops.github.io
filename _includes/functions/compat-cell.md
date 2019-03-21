{% case include.state %}
  {% when "true" %}{{yes}}
  {% when "false" %}{{no}}
  {% when "na" %}
  {% when "untested" %}?
  {% else %}{% include ERROR_43_unexpected_value %}
{% endcase %}
