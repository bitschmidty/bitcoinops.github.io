---
layout: page
title: Compatibility Comparison
permalink: /en/compatibility/
foo: bar
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
    <th></th>
    <th colspan="5">Receiving support</th>
    <th colspan="5">Sending support</th>
  </tr>
  <tr>
    <th>Tool</th>
    <th>Notification</th>
    <th>List</th>
    <th>Details</th>
    <th>Shows replaced</th>
    <th>Shows original</th>
    <th>Signals BIP125</th>
    <th>List</th>
    <th>Details</th>
    <th>Shows replaced</th>
    <th>Shows original</th>
  </tr>

{% assign tools = site.data.compatibility | sort %}
{% for wrapped_tool in tools %}
  {% assign tool = wrapped_tool[1] %}
  <tr>
    <td class="left"><a href="{{tool.internal_url}}">{{tool.name}}</a></td>
    <td><a href="{{tool.internal_url}}#receive-notification">{% include functions/compat-cell.md state=tool.rbf.features.receive.notification %}</a></td>
    <td><a href="{{tool.internal_url}}#receive-list">{% include functions/compat-cell.md state=tool.rbf.features.receive.list %}</a></td>
    <td><a href="{{tool.internal_url}}#receive-details">{% include functions/compat-cell.md state=tool.rbf.features.receive.details %}</a></td>
    <td><a href="{{tool.internal_url}}#receive-replaced">{% include functions/compat-cell.md state=tool.rbf.features.receive.shows_replaced_version %}</a></td>
    <td><a href="{{tool.internal_url}}#receive-replaced">{% include functions/compat-cell.md state=tool.rbf.features.receive.shows_original_version %}</a></td>
    <td><a href="{{tool.internal_url}}#send-signals_bip125">{% include functions/compat-cell.md state=tool.rbf.features.send.signals_bip125 %}</a></td>
    <td><a href="{{tool.internal_url}}#send-list">{% include functions/compat-cell.md state=tool.rbf.features.send.list %}</a></td>
    <td><a href="{{tool.internal_url}}#send-details">{% include functions/compat-cell.md state=tool.rbf.features.send.details %}</a></td>
    <td><a href="{{tool.internal_url}}#send-replaced">{% include functions/compat-cell.md state=tool.rbf.features.send.shows_replaced_version %}</a></td>
    <td><a href="{{tool.internal_url}}#send-replaced">{% include functions/compat-cell.md state=tool.rbf.features.send.shows_original_version %}</a></td>
  </tr>
{% endfor %}

</table>
