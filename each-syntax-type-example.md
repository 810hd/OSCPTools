---
description: I listed all of the formatting capabilities on gitbook here.
---

# \*syntax\*

## Heading 1

## Heading 1.1

### Heading 2

### Heading 2.1

#### Heading 3

#### Heading 3.1

* one
  * one point one
  * one point two
    * one point two point one
* two
* three

1. Plan
2. Enumeration
3. Local Privilege Escalation
4. Root Privilege Escalation

* [ ] plan
* [x] enumerate
* [ ] local privilege escalation
* [x] root privilege escalation

```text
cat codeBlock.txt
#This is an awesome feature to display exploits and stuff
Will come in handy for examples. Or just a screenshot but use this to easily copy
```

> Four sscore and seven years ago Lil Uzi Vert changed my life. This is verbatim what Lincoln said in Gettysburg, PA in 1863

![caption n ting](.gitbook/assets/2%20%281%29.png)

| X values | Y values | Z values |
| :--- | :--- | :--- |
| 0 | 2 | 10 |
| 1 | 4 | 20 |

{% hint style="info" %}
here is a hint. take it
{% endhint %}

{% hint style="warning" %}
warning
{% endhint %}

{% hint style="danger" %}
danger
{% endhint %}

{% hint style="success" %}
we got it
{% endhint %}

{% page-ref page="./" %}

{% api-method method="get" host="" path="" %}
{% api-method-summary %}
title?
{% endapi-method-summary %}

{% api-method-description %}

{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="" type="string" required=false %}

{% endapi-method-parameter %}
{% endapi-method-path-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```

```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

{% tabs %}
{% tab title="First Tab" %}
tab 1 info
{% endtab %}

{% tab title="Second Tab" %}
tab 2 info
{% endtab %}
{% endtabs %}

$$
a = b +c
$$

{% file src=".gitbook/assets/linux\_architecture.png" caption="here is a file" %}



