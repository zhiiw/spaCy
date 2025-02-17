---
title: SpanResolver
tag: class,experimental
source: spacy-experimental/coref/span_resolver_component.py
teaser: 'Pipeline component for resolving tokens into spans'
api_base_class: /api/pipe
api_string_name: span_resolver
api_trainable: true
---

> #### Installation
>
> ```bash
> $ pip install -U spacy-experimental
> ```

<Infobox title="Important note" variant="warning">

This component not yet integrated into spaCy core, and is available via the
extension package
[`spacy-experimental`](https://github.com/explosion/spacy-experimental) starting
in version 0.6.0. It exposes the component via
[entry points](/usage/saving-loading/#entry-points), so if you have the package
installed, using `factory = "experimental_span_resolver"` in your
[training config](/usage/training#config) or
`nlp.add_pipe("experimental_span_resolver")` will work out-of-the-box.

</Infobox>

A `SpanResolver` component takes in tokens (represented as `Span` objects of
length 1) and resolves them into `Span` objects of arbitrary length. The initial
use case is as a post-processing step on word-level
[coreference resolution](/api/coref). The input and output keys used to store
`Span` objects are configurable.

## Assigned Attributes {#assigned-attributes}

Predictions will be saved to `Doc.spans` as [`SpanGroup`s](/api/spangroup).

Input token spans will be read in using an input prefix, by default
`"coref_head_clusters"`, and output spans will be saved using an output prefix
(default `"coref_clusters"`) plus a serial number starting from one. The
prefixes are configurable.

| Location                                          | Value                                                                     |
| ------------------------------------------------- | ------------------------------------------------------------------------- |
| `Doc.spans[output_prefix + "_" + cluster_number]` | One group of predicted spans. Cluster number starts from 1. ~~SpanGroup~~ |

## Config and implementation {#config}

The default config is defined by the pipeline component factory and describes
how the component should be configured. You can override its settings via the
`config` argument on [`nlp.add_pipe`](/api/language#add_pipe) or in your
[`config.cfg` for training](/usage/training#config). See the
[model architectures](/api/architectures#coref-architectures) documentation for
details on the architectures and their arguments and hyperparameters.

> #### Example
>
> ```python
> from spacy_experimental.coref.span_resolver_component import DEFAULT_SPAN_RESOLVER_MODEL
> from spacy_experimental.coref.coref_util import DEFAULT_CLUSTER_PREFIX, DEFAULT_CLUSTER_HEAD_PREFIX
> config={
>     "model": DEFAULT_SPAN_RESOLVER_MODEL,
>     "input_prefix": DEFAULT_CLUSTER_HEAD_PREFIX,
>     "output_prefix": DEFAULT_CLUSTER_PREFIX,
> },
> nlp.add_pipe("experimental_span_resolver", config=config)
> ```

| Setting         | Description                                                                                                                                            |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `model`         | The [`Model`](https://thinc.ai/docs/api-model) powering the pipeline component. Defaults to [SpanResolver](/api/architectures#SpanResolver). ~~Model~~ |
| `input_prefix`  | The prefix to use for input `SpanGroup`s. Defaults to `coref_head_clusters`. ~~str~~                                                                   |
| `output_prefix` | The prefix for predicted `SpanGroup`s. Defaults to `coref_clusters`. ~~str~~                                                                           |

## SpanResolver.\_\_init\_\_ {#init tag="method"}

> #### Example
>
> ```python
> # Construction via add_pipe with default model
> span_resolver = nlp.add_pipe("experimental_span_resolver")
>
> # Construction via add_pipe with custom model
> config = {"model": {"@architectures": "my_span_resolver.v1"}}
> span_resolver = nlp.add_pipe("experimental_span_resolver", config=config)
>
> # Construction from class
> from spacy_experimental.coref.span_resolver_component import SpanResolver
> span_resolver = SpanResolver(nlp.vocab, model)
> ```

Create a new pipeline instance. In your application, you would normally use a
shortcut for this and instantiate the component using its string name and
[`nlp.add_pipe`](/api/language#add_pipe).

| Name            | Description                                                                                         |
| --------------- | --------------------------------------------------------------------------------------------------- |
| `vocab`         | The shared vocabulary. ~~Vocab~~                                                                    |
| `model`         | The [`Model`](https://thinc.ai/docs/api-model) powering the pipeline component. ~~Model~~           |
| `name`          | String name of the component instance. Used to add entries to the `losses` during training. ~~str~~ |
| _keyword-only_  |                                                                                                     |
| `input_prefix`  | The prefix to use for input `SpanGroup`s. Defaults to `coref_head_clusters`. ~~str~~                |
| `output_prefix` | The prefix for predicted `SpanGroup`s. Defaults to `coref_clusters`. ~~str~~                        |

## SpanResolver.\_\_call\_\_ {#call tag="method"}

Apply the pipe to one document. The document is modified in place and returned.
This usually happens under the hood when the `nlp` object is called on a text
and all pipeline components are applied to the `Doc` in order. Both
[`__call__`](#call) and [`pipe`](#pipe) delegate to the [`predict`](#predict)
and [`set_annotations`](#set_annotations) methods.

> #### Example
>
> ```python
> doc = nlp("This is a sentence.")
> span_resolver = nlp.add_pipe("experimental_span_resolver")
> # This usually happens under the hood
> processed = span_resolver(doc)
> ```

| Name        | Description                      |
| ----------- | -------------------------------- |
| `doc`       | The document to process. ~~Doc~~ |
| **RETURNS** | The processed document. ~~Doc~~  |

## SpanResolver.pipe {#pipe tag="method"}

Apply the pipe to a stream of documents. This usually happens under the hood
when the `nlp` object is called on a text and all pipeline components are
applied to the `Doc` in order. Both [`__call__`](/api/span-resolver#call) and
[`pipe`](/api/span-resolver#pipe) delegate to the
[`predict`](/api/span-resolver#predict) and
[`set_annotations`](/api/span-resolver#set_annotations) methods.

> #### Example
>
> ```python
> span_resolver = nlp.add_pipe("experimental_span_resolver")
> for doc in span_resolver.pipe(docs, batch_size=50):
>     pass
> ```

| Name           | Description                                                   |
| -------------- | ------------------------------------------------------------- |
| `stream`       | A stream of documents. ~~Iterable[Doc]~~                      |
| _keyword-only_ |                                                               |
| `batch_size`   | The number of documents to buffer. Defaults to `128`. ~~int~~ |
| **YIELDS**     | The processed documents in order. ~~Doc~~                     |

## SpanResolver.initialize {#initialize tag="method"}

Initialize the component for training. `get_examples` should be a function that
returns an iterable of [`Example`](/api/example) objects. **At least one example
should be supplied.** The data examples are used to **initialize the model** of
the component and can either be the full training data or a representative
sample. Initialization includes validating the network,
[inferring missing shapes](https://thinc.ai/docs/usage-models#validation) and
setting up the label scheme based on the data. This method is typically called
by [`Language.initialize`](/api/language#initialize).

> #### Example
>
> ```python
> span_resolver = nlp.add_pipe("experimental_span_resolver")
> span_resolver.initialize(lambda: examples, nlp=nlp)
> ```

| Name           | Description                                                                                                                                                                |
| -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `get_examples` | Function that returns gold-standard annotations in the form of [`Example`](/api/example) objects. Must contain at least one `Example`. ~~Callable[[], Iterable[Example]]~~ |
| _keyword-only_ |                                                                                                                                                                            |
| `nlp`          | The current `nlp` object. Defaults to `None`. ~~Optional[Language]~~                                                                                                       |

## SpanResolver.predict {#predict tag="method"}

Apply the component's model to a batch of [`Doc`](/api/doc) objects, without
modifying them. Predictions are returned as a list of `MentionClusters`, one for
each input `Doc`. A `MentionClusters` instance is just a list of lists of pairs
of `int`s, where each item corresponds to an input `SpanGroup`, and the `int`s
correspond to token indices.

> #### Example
>
> ```python
> span_resolver = nlp.add_pipe("experimental_span_resolver")
> spans = span_resolver.predict([doc1, doc2])
> ```

| Name        | Description                                                   |
| ----------- | ------------------------------------------------------------- |
| `docs`      | The documents to predict. ~~Iterable[Doc]~~                   |
| **RETURNS** | The predicted spans for the `Doc`s. ~~List[MentionClusters]~~ |

## SpanResolver.set_annotations {#set_annotations tag="method"}

Modify a batch of documents, saving predictions using the output prefix in
`Doc.spans`.

> #### Example
>
> ```python
> span_resolver = nlp.add_pipe("experimental_span_resolver")
> spans = span_resolver.predict([doc1, doc2])
> span_resolver.set_annotations([doc1, doc2], spans)
> ```

| Name    | Description                                                   |
| ------- | ------------------------------------------------------------- |
| `docs`  | The documents to modify. ~~Iterable[Doc]~~                    |
| `spans` | The predicted spans for the `docs`. ~~List[MentionClusters]~~ |

## SpanResolver.update {#update tag="method"}

Learn from a batch of [`Example`](/api/example) objects. Delegates to
[`predict`](/api/span-resolver#predict).

> #### Example
>
> ```python
> span_resolver = nlp.add_pipe("experimental_span_resolver")
> optimizer = nlp.initialize()
> losses = span_resolver.update(examples, sgd=optimizer)
> ```

| Name           | Description                                                                                                              |
| -------------- | ------------------------------------------------------------------------------------------------------------------------ |
| `examples`     | A batch of [`Example`](/api/example) objects to learn from. ~~Iterable[Example]~~                                        |
| _keyword-only_ |                                                                                                                          |
| `drop`         | The dropout rate. ~~float~~                                                                                              |
| `sgd`          | An optimizer. Will be created via [`create_optimizer`](#create_optimizer) if not set. ~~Optional[Optimizer]~~            |
| `losses`       | Optional record of the loss during training. Updated using the component name as the key. ~~Optional[Dict[str, float]]~~ |
| **RETURNS**    | The updated `losses` dictionary. ~~Dict[str, float]~~                                                                    |

## SpanResolver.create_optimizer {#create_optimizer tag="method"}

Create an optimizer for the pipeline component.

> #### Example
>
> ```python
> span_resolver = nlp.add_pipe("experimental_span_resolver")
> optimizer = span_resolver.create_optimizer()
> ```

| Name        | Description                  |
| ----------- | ---------------------------- |
| **RETURNS** | The optimizer. ~~Optimizer~~ |

## SpanResolver.use_params {#use_params tag="method, contextmanager"}

Modify the pipe's model, to use the given parameter values. At the end of the
context, the original parameters are restored.

> #### Example
>
> ```python
> span_resolver = nlp.add_pipe("experimental_span_resolver")
> with span_resolver.use_params(optimizer.averages):
>     span_resolver.to_disk("/best_model")
> ```

| Name     | Description                                        |
| -------- | -------------------------------------------------- |
| `params` | The parameter values to use in the model. ~~dict~~ |

## SpanResolver.to_disk {#to_disk tag="method"}

Serialize the pipe to disk.

> #### Example
>
> ```python
> span_resolver = nlp.add_pipe("experimental_span_resolver")
> span_resolver.to_disk("/path/to/span_resolver")
> ```

| Name           | Description                                                                                                                                |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| `path`         | A path to a directory, which will be created if it doesn't exist. Paths may be either strings or `Path`-like objects. ~~Union[str, Path]~~ |
| _keyword-only_ |                                                                                                                                            |
| `exclude`      | String names of [serialization fields](#serialization-fields) to exclude. ~~Iterable[str]~~                                                |

## SpanResolver.from_disk {#from_disk tag="method"}

Load the pipe from disk. Modifies the object in place and returns it.

> #### Example
>
> ```python
> span_resolver = nlp.add_pipe("experimental_span_resolver")
> span_resolver.from_disk("/path/to/span_resolver")
> ```

| Name           | Description                                                                                     |
| -------------- | ----------------------------------------------------------------------------------------------- |
| `path`         | A path to a directory. Paths may be either strings or `Path`-like objects. ~~Union[str, Path]~~ |
| _keyword-only_ |                                                                                                 |
| `exclude`      | String names of [serialization fields](#serialization-fields) to exclude. ~~Iterable[str]~~     |
| **RETURNS**    | The modified `SpanResolver` object. ~~SpanResolver~~                                            |

## SpanResolver.to_bytes {#to_bytes tag="method"}

> #### Example
>
> ```python
> span_resolver = nlp.add_pipe("experimental_span_resolver")
> span_resolver_bytes = span_resolver.to_bytes()
> ```

Serialize the pipe to a bytestring.

| Name           | Description                                                                                 |
| -------------- | ------------------------------------------------------------------------------------------- |
| _keyword-only_ |                                                                                             |
| `exclude`      | String names of [serialization fields](#serialization-fields) to exclude. ~~Iterable[str]~~ |
| **RETURNS**    | The serialized form of the `SpanResolver` object. ~~bytes~~                                 |

## SpanResolver.from_bytes {#from_bytes tag="method"}

Load the pipe from a bytestring. Modifies the object in place and returns it.

> #### Example
>
> ```python
> span_resolver_bytes = span_resolver.to_bytes()
> span_resolver = nlp.add_pipe("experimental_span_resolver")
> span_resolver.from_bytes(span_resolver_bytes)
> ```

| Name           | Description                                                                                 |
| -------------- | ------------------------------------------------------------------------------------------- |
| `bytes_data`   | The data to load from. ~~bytes~~                                                            |
| _keyword-only_ |                                                                                             |
| `exclude`      | String names of [serialization fields](#serialization-fields) to exclude. ~~Iterable[str]~~ |
| **RETURNS**    | The `SpanResolver` object. ~~SpanResolver~~                                                 |

## Serialization fields {#serialization-fields}

During serialization, spaCy will export several data fields used to restore
different aspects of the object. If needed, you can exclude them from
serialization by passing in the string names via the `exclude` argument.

> #### Example
>
> ```python
> data = span_resolver.to_disk("/path", exclude=["vocab"])
> ```

| Name    | Description                                                    |
| ------- | -------------------------------------------------------------- |
| `vocab` | The shared [`Vocab`](/api/vocab).                              |
| `cfg`   | The config file. You usually don't want to exclude this.       |
| `model` | The binary model data. You usually don't want to exclude this. |
