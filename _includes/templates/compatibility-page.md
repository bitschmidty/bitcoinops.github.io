{{tool.description}}

## Replace-by-Fee (RBF)

An unconfirmed transaction can be replaced by another version of the
same transaction that spends the same inputs.  Most full nodes support
this if the earlier transaction enables [BIP125](https://github.com/bitcoin/bips/blob/master/bip-0125.mediawiki) signaling and the
replacement transaction increases the amount of fee paid.  In terms of
block chain space used, this is the most efficient form of fee bumping.

### Receiving support

{% assign rbf = tool.rbf.features. %}
{% case rbf.receive.notification %}
  {% when "true" %}{:.feature-good}
  - **Notifies on replacement**<br>
    Sends a notification about replaced transactions
  {% when "false" %}{:.feature-bad}
  - **No notification**<br>
    Doesn't tell you when a transaction has been replaced
  {% when "na" %}{:.feature-neutral}
  - **No notification**<br>
    There are no notifications for this wallet
  {% when "untested" %}{:.feature-neutral}
  - **No replacement tagging**<br>
    We didn’t test this
  {% else %}{% include ERROR_42_UNEXPECTED_VALUE %}
{% endcase %}

{% case rbf.receive.list %}
  {% when "true" %}{:.feature-good}
  - **Tags transaction as replacable**<br>
    Indicates that a transaction opts into BIP125
  {% when "false" %}{:.feature-bad}
  - **BIP125 unclear**<br>
    Doesn't tell users transactions opt-in to BIP125
  {% when "na" %}{:.feature-neutral}
  - **This wallet doesn't handle unconfirmed transactions**<br>
    This wallet doesn't indicate tranacitons opt-in to BIP125 because it
    only deals with confirmed transactions
  {% when "untested" %}{:.feature-neutral}
  - **BIP125 tagging**<br>
    We didn’t test this
  {% else %}{% include ERROR_42_UNEXPECTED_VALUE %}
{% endcase %}

{% case rbf.receive.details %}
  {% when "true" %}{:.feature-good}
  - **BIP125 in details**<br>
    FIXME
  {% when "false" %}{:.feature-bad}
  - **No BIP125 in details**<br>
    FIXME
  {% when "na" %}{:.feature-neutral}
  - **FIXME**<br>
    FIXME
  {% when "untested" %}{:.feature-neutral}
  - **FIXME**<br>
    We didn’t test this
  {% else %}{% include ERROR_42_UNEXPECTED_VALUE %}
{% endcase %}

{% case rbf.receive.shows_replaced_version %}
  {% when "true" %}{:.feature-good}
  - **Shows replacement version**<br>
    FIXME
  {% when "false" %}{:.feature-bad}
  - **Doesn't show replacement transactions**<br>
    FIXME
  {% when "na" %}{:.feature-neutral}
  - **FIXME**<br>
    FIXME
  {% when "untested" %}{:.feature-neutral}
  - **FIXME**<br>
    We didn’t test this
  {% else %}{% include ERROR_42_UNEXPECTED_VALUE %}
{% endcase %}

{% case rbf.receive.shows_original_version %}
  {% when "true" %}{:.feature-good}
  - **Shows replaced transactions**<br>
    FIXME
  {% when "false" %}{:.feature-bad}
  - **Doesn't show replaced versions**<br>
    FIXME
  {% when "na" %}{:.feature-neutral}
  - **FIXME**<br>
    FIXME
  {% when "untested" %}{:.feature-neutral}
  - **FIXME**<br>
    We didn’t test this
  {% else %}{% include ERROR_42_UNEXPECTED_VALUE %}
{% endcase %}

### Sending support

{% case rbf.send.signals_bip125 %}
  {% when "true" %}{:.feature-good}
  - **Signals BIP125 by default**<br>
    FIXME
  {% when "false" %}{:.feature-bad}
  - **Doesn't signal BIP125**<br>
    FIXME
  {% when "na" %}{:.feature-neutral}
  - **FIXME**<br>
    FIXME
  {% when "untested" %}{:.feature-neutral}
  - **FIXME**<br>
    We didn’t test this
  {% else %}{% include ERROR_42_UNEXPECTED_VALUE %}
{% endcase %}

{% case rbf.send.list %}
  {% when "true" %}{:.feature-good}
  - **Notes transactions are replaceable in list**<br>
    FIXME
  {% when "false" %}{:.feature-bad}
  - **Doesn't indicate transactions are replacable**<br>
    FIXME
  {% when "na" %}{:.feature-neutral}
  - **FIXME**<br>
    FIXME
  {% when "untested" %}{:.feature-neutral}
  - **FIXME**<br>
    We didn’t test this
  {% else %}{% include ERROR_42_UNEXPECTED_VALUE %}
{% endcase %}

{% case rbf.send.details %}
  {% when "true" %}{:.feature-good}
  - **Replacability info in details view**<br>
    FIXME
  {% when "false" %}{:.feature-bad}
  - **No replacability info in details view**<br>
    FIXME
  {% when "na" %}{:.feature-neutral}
  - **FIXME**<br>
    FIXME
  {% when "untested" %}{:.feature-neutral}
  - **FIXME**<br>
    We didn’t test this
  {% else %}{% include ERROR_42_UNEXPECTED_VALUE %}
{% endcase %}

{% case rbf.send.shows_replaced_version %}
  {% when "true" %}{:.feature-good}
  - **Shows replacement transactions**<br>
    FIXME
  {% when "false" %}{:.feature-bad}
  - **Doesn't show replacement transactions**<br>
    FIXME
  {% when "na" %}{:.feature-neutral}
  - **FIXME**<br>
    FIXME
  {% when "untested" %}{:.feature-neutral}
  - **FIXME**<br>
    We didn’t test this
  {% else %}{% include ERROR_42_UNEXPECTED_VALUE %}
{% endcase %}

{% case rbf.send.shows_original_version %}
  {% when "true" %}{:.feature-good}
  - **Shows replaced transactions**<br>
    FIXME
  {% when "false" %}{:.feature-bad}
  - **Doesn't show replaced transactions**<br>
    FIXME
  {% when "na" %}{:.feature-neutral}
  - **FIXME**<br>
    FIXME
  {% when "untested" %}{:.feature-neutral}
  - **FIXME**<br>
    We didn’t test this
  {% else %}{% include ERROR_42_UNEXPECTED_VALUE %}
{% endcase %}

### Usability

*Click on a thumbnail for a larger image or to the play its video.*

{% for example in tool.rbf.examples %}{% capture /dev/null %}
  {% if example.link %}
    {% assign link = example.link %}
  {% else %}
    {% assign link = example.image %}
  {% endif %}
{% endcapture %}
<div markdown="1" style="max-width: 300px; float: left; padding: 20px;">
[![{{example.caption|escape_once}}]({{example.image}}){:width="250px"}]({{link}})
<br />{{example.caption}}</div>
{% endfor %}
