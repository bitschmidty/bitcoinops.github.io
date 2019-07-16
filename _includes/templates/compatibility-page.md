{{tool.description}}

## Replace-by-Fee (RBF)

An unconfirmed transaction can be replaced by another version of the
same transaction that spends the same inputs.  Most full nodes support
this if the earlier transaction enables [BIP125](https://github.com/bitcoin/bips/blob/master/bip-0125.mediawiki) signaling and the
replacement transaction increases the amount of fee paid.  In terms of
block chain space used, this is the most efficient form of fee bumping.

### Support

{% assign notification = tool.rbf.features.receive.notification %}
{% if notification == "true" %}
  {:.feature-good}
  - **Notifies on replacement**<br>
    Sends a notification about replaced transactions
{% elsif notification == "false" %}
  {:.feature-bad}
  - **No notification**<br>
    Doesn't tell you when a transaction has been replaced
{% elsif notification == "na" %}
  {:.feature-neutral}
  - **No notification**<br>
    There are no notifications for this wallet
{% else %}
  {:.feature-neutral}
  - **No replacement tagging**<br>
    We didn’t test this 
{% endif %}   

{% if tool.rbf.list == true %}
  {:.feature-good}
  - **Tags replacable transactions**<br>
    Notes which transactions are BIP125 replacable
{% else %}
  {:.feature-bad}
  - **No replacement tagging**<br>
    Doesn't indicate which transactions opt-in to RBF
{% endif %} 
  
{% if tool.rbf.details == true %}
  {:.feature-good}
  - **Detailed replacement information**<br>
    Information about replacements contained in transaction details view
{% else %}
  {:.feature-bad}
  - **No extra information**<br>
    Users can't access additional information about replacements
{% endif %}

{% if tool.rbf.bumped_versions == true %}
  {:.feature-good}
  - **Replacements record**<br>
    Keeps a record of all replacement transactions
{% else %}
  {:.feature-bad}
  - **No replacement record**<br>
    After any transaction confirms, all information about other versions is lost
{% endif %}

{% if tool.rbf.send == true %}
  {:.feature-good}
  - **Opts in to BIP125 RBF**<br>
    This wallet can easily replace its own unconfirmed transactions
{% else %}
  {:.feature-bad}
  - **Does not use BIP125 opt-in RBF**<br>
    This wallet can't reliably fee bump its own transactions
{% endif %}

### Usability

{% for example in tool.rbf.examples %}{% capture /dev/null %}
  {% if example.link %}
    {% assign link = example.link %}
  {% else %}
    {% assign link = example.image %}
  {% endif %}
{% endcapture %}
<br />[![{{example.caption|escape_once}}]({{example.image}}){:width="250px"}]({{link}})
<br />{{example.caption}}
{% endfor %}
