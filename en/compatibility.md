---
layout: page
title: Compatibility Comparison
permalink: /en/compatibility/
---
{% assign yes = '<span class="feature-good"><strong>✓</strong></span>' %}
{% assign no = '<span class="feature-bad"><strong>✕</strong></span>' %}
<style>
th, td { text-align: center; }
td.left { text-align: left; }
</style>

## Replace-by-Fee (RBF)

<table>
  <tr>
    <th title="Name of the tool">Tool</th>
    <th title="Shows notifications about replacements">Notification</th>
    <th title="Tags BIP125 tranactions in list view">List</th>
    <th title="Shows details about replacable transactions">Details</th>
    <th title="Shows old tx versions">Versions</th>
    <th title="Allows sending BIP125 transactions">Send</th>
  </tr>

{% for wrapped_tool in site.data.compatibility %}
  {% assign tool = wrapped_tool[1] %}
  <tr>
    <td class="left"><a href="{{tool.internal_url}}">{{tool.name}}</a></td>
    <td>{% if tool.rbf.notification    == true %}{{yes}}{% else %}{{no}}{% endif %}</td>
    <td>{% if tool.rbf.list            == true %}{{yes}}{% else %}{{no}}{% endif %}</td>
    <td>{% if tool.rbf.details         == true %}{{yes}}{% else %}{{no}}{% endif %}</td>
    <td>{% if tool.rbf.bumped_versions == true %}{{yes}}{% else %}{{no}}{% endif %}</td>
    <td>{% if tool.rbf.send            == true %}{{yes}}{% else %}{{no}}{% endif %}</td>
  </tr>
{% endfor %}

</table>