# Content Negotiation

The API system has builtin [content negotiation](https://en.wikipedia.org/wiki/Content_negotiation) capabilities.
It leverages the [`willdurand/negotiation`](https://github.com/willdurand/Negotiation) library.

By default, only the [JSON-LD](https://json-ld.org) format is enabled. However API Platform Core supports many more formats and can be extended.

The framework natively supports JSON-LD, HAL, raw JSON, XML, YAML and CSV (YAML and CSV support is only available if you use Symfony 3.2+).

Both XML and JSON formats are experimental and there are no assurance that we will not break them.

API Platform Core will automatically detect the best resolving format depending on:

* enabled formats (link to docs for this / see below)
* the `Accept` HTTP header
* Alternatively to send the proper `Accept` HTTP header, you can also request a specific format by adding its name as the extension of the URL.
* Available formats are :

Format                                                          | Format name  | MIME types                    | Backward Compatibility guaranteed
----------------------------------------------------------------|--------------|-------------------------------|----------------------------------------
[JSON-LD](https://json-ld.org)                                  | `jsonld`     | `application/ld+json`         | yes
[HAL](http://stateless.co/hal_specification.html)               | `jsonhal`    | `application/hal+json`        | yes
JSON                                                            | `json`       |  `application/json`           | no
XML                                                             | `xml`        | `application/xml`, `text/xml` | no
HTML (API docs)                                                 | `html`       | `text/html`                   | no


If the client requested format is not specified (if it's not supported, it will throw an HTTP bad format error), the response format will be the first format defined in the `formats` configuration key (see below).
An example using the builtin XML support is available in Behat specs: https://github.com/api-platform/core/blob/master/features/content_negotiation.feature


The API Platform content negotiation system is extensible. Support for other formats (such as [JSONAPI](http://jsonapi.org/))
can be added by [creating and registering appropriate encoders and, sometimes, normalizers](https://symfony.com/doc/current/serializer.html#adding-normalizers-and-encoders). Adding support for other
standard hypermedia formats upstream is very welcome. Don't hesitate to contribute by adding your encoders and normalizers
to API Platform Core.


## Enabling Several Formats

The first required step is to configure allowed formats. The following configuration will enable the support of XML (built-in)
and of a custom format called `myformat` and having `application/vnd.myformat` as [MIME type](https://en.wikipedia.org/wiki/Media_type).

```yaml
# app/config/config.yml

api_platform:
    # ...
    formats:
        jsonld:   ['application/ld+json']
        jsonhal:  ['application/hal+json']
        json:     ['application/json']
        xml:      ['application/xml', 'text/xml']
        html:     ['text/html']
        myformat: ['application/vnd.myformat']
```

Because the Symfony Serializer component is able to serialize objects in XML, sending an `Accept` HTTP header with the
`text/xml` string as value is enough to retrieve XML documents from our API. However API Platform knows nothing about the
`myformat` format. We need to register an encoder and optionally a normalizer for this format.

## Registering a Custom Serializer

If you are adding support for a format not supported by default by API Platform nor by the Symfony Serializer Component,
you need to create custom encoder, decoder and eventually a normalizer and a denormalizer. Refer to the
Symfony documentation to learn [how to create and register such classes](https://symfony.com/doc/current/cookbook/serializer.html#adding-normalizers-and-encoders).

API Platform Core will automatically call the serializer with your defined format name (`myformat` in previous examples)
as `format` parameter during the deserialization process. Then it will return the result to the client with the asked MIME
type using its built-in responder.

Previous chapter: [The Event System](events.md)

Next chapter: [Using External JSON-LD Vocabularies](external-vocabularies.md)