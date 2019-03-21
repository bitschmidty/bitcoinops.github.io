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

{% for wrapped_tool in site.data.compatibility %}
  {% assign tool = wrapped_tool[1] %}
  <tr>
    <td class="left"><a href="{{tool.internal_url}}">{{tool.name}}</a></td>
    <td>{% include functions/compat-cell.md state=tool.rbf.features.receive.notification %}</td>
    <td>{% include functions/compat-cell.md state=tool.rbf.features.receive.list %}</td>
    <td>{% include functions/compat-cell.md state=tool.rbf.features.receive.details %}</td>
    <td>{% include functions/compat-cell.md state=tool.rbf.features.receive.shows_replaced_version %}</td>
    <td>{% include functions/compat-cell.md state=tool.rbf.features.receive.shows_original_version %}</td>
    <td>{% include functions/compat-cell.md state=tool.rbf.features.send.signals_bip125 %}</td>
    <td>{% include functions/compat-cell.md state=tool.rbf.features.send.list %}</td>
    <td>{% include functions/compat-cell.md state=tool.rbf.features.send.details %}</td>
    <td>{% include functions/compat-cell.md state=tool.rbf.features.send.shows_replaced_version %}</td>
    <td>{% include functions/compat-cell.md state=tool.rbf.features.send.shows_original_version %}</td>
  </tr>
{% endfor %}

</table>
