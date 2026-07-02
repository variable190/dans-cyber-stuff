# Insecure Deserialisation

## What is Insecure Deserialisation?

Insecure Deserialisation occurs when an application deserializes untrusted data without sufficient validation.

## Exploits

- [PHP Phar Deserialisation](insecure-deserialisation/phar-deserialisation.md)
- [Java Gadget Chains](insecure-deserialisation/java-gadgets.md)

## Impact

RCE.

## Prevention

Avoid untrusted deserial. Use safe formats.

## Tools & Payloads

ysoserial, phar tools.
