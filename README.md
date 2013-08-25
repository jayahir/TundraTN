# TundraTN ❄

A package of useful services for webMethods Trading Networks 7.1 or higher.

## Dependencies

TundraTN is dependent on the following packages:

* Tundra - https://github.com/Permafrost/Tundra.git
* WmTN

## Installation

You have two choices for installing TundraTN: git or zip. If you are comfortable using git,
I recommend this method as you can then easily switch between package versions using git
checkout and download new versions using git fetch.

### Using Git

To install with this method, first make sure that:
* Git is [installed](http://git-scm.com/downloads) on your Integration Server
* Your Integration Server has internet access to https://github.com (for cloning the repository)
* The dependent packages listed above are installed and enabled on your Integration Server

From your Integration Server installation:

```sh
$ cd ./packages/
$ git clone https://github.com/Permafrost/TundraTN.git
$ cd ./TundraTN/
$ git checkout v<n.n.n> # where <n.n.n> is the desired version
```

Then activate and enable the TundraTN package from the package management web page on your Integration Server
web administration site.

### Using Zip

1. Download a zip of the desired version of the package from https://github.com/Permafrost/TundraTN/releases
2. Copy the TundraTN-n.n.n.zip file to your Integration Server's ./replicate/inbound/ directory
3. Install and activate the TundraTN package release (TundraTN-n.n.n.zip) from the package management web page
on your Integration Server's web administration site

## Upgrading

When upgrading you have to choose the same method used to originally install the package. Unfortunately, if git
wasn't used to install the package then you can't use git to upgrade it either. However, if you want to switch
to using git to manage the package, delete the installed package and start over using the git method for
installation.

### Using Git

To upgrade with this method, first make sure that:
* Git is [installed](http://git-scm.com/downloads) on your Integration Server
* Your Integration Server has internet access to https://github.com (for fetching updates from the repository)
* The dependent packages listed above are installed and enabled on your Integration Server
* You originally installed TundraTN using the git method described above

From your Integration Server installation:

```sh
$ cd ./packages/TundraTN/
$ git fetch
$ git checkout v<n.n.n> # where <n.n.n> is the desired updated version
```

Then reload the TundraTN package from the package management web page on your Integration Server web administration
site.

### Using Zip

1. Download a zip of the desired updated version of the package from https://github.com/Permafrost/TundraTN/releases
2. Copy the TundraTN-n.n.n.zip file to your Integration Server's ./replicate/inbound/ directory
3. Install and activate the updated TundraTN package release (TundraTN-n.n.n.zip) from the package management web page
on your Integration Server's web administration site

## Conventions

1. All input and output pipeline arguments are prefixed with '$' as a poor-man's
   scoping mechanism (typical user-space variables will be unprefixed), except
   Trading Networks-specific arguments, such as bizdoc, sender and receiver
2. All boolean arguments are suffixed with a '?'
3. Single-word argument names are preferred. Where multiple words are necessary,
   words are separated with a '.'
4. Service namespace is kept flat. Namespace folders are usually nouns. Service
   names are usually verbs, indicating the action performed on the noun (parent
   folder)
5. All private elements are kept in the tundra.tn.support folder. All other
   elements comprise the public API of the package. As the private
   elements do not contribute to the public API, they are liable to change at
   any time. Enter at your own risk

## Tests

*Almost* every service in TundraTN has unit tests, located in the
tundra.tn.support.test folder.

To run the test suite, either:
* run tundra:test($package = "TundraTN") service directly
* visit <http://localhost:5555/invoke/tundra/test?$package=TundraTN>
  (substitute your own Integration Server host and port for localhost:5555)

## Services

Top-level services for the most common tasks:

```java
// Edits the given XML or flat file bizdoc content part with the list of {key, value} pairs
// specified in $amendments.
//
// The keys in $amendments can be fully-qualified (for example, "a/b/c[0]"), and the values can
// include percent-delimited variable substitution strings which will be substituted prior to
// being inserted in $document.
//
// The bizdoc user status is first updated to 'AMENDING', then the content part identified by
// $part.input (or the default content part if not specified) is parsed to an IData document
// using the named $schema (or the schema configured on the TN document type if not specified),
// the amendments are applied via the $amendments[] {key, value} pairs, the amended IData
// document is then emitted as stream then added to the bizdoc as a new content part identified
// by $part.output (or 'amendment' if not specified), and the bizdoc user status is updated to
// 'AMENDED'.
//
// This service is designed to be used in conjunction with other TN processing rule actions, such
// as the 'Deliver document by' action, which can use the amended content part for delivery
// rather than the original content part.
//
// Supports 'strict' mode processing of bizdocs: if any $strict error classes are set to 'true' and
// the bizdoc contains errors for any of these classes, the bizdoc will not be processed; instead an
// exception will be thrown and handled by the $catch service. For example, if you have enabled
// duplicate document checking on the Trading Networks document type and do not wish to process
// duplicates, set the $strict/Saving error class to 'true' and duplicate documents will not
// be processed and will instead have their user status set to 'ERROR' (when using the standard
// $catch service).
tundra.tn:amend(bizdoc, $amendments[], $catch, $finally, $schema, $part.input, $part.output, $encoding.input, $encoding.output, $strict);

// Processes a Trading Networks document by parsing the given document content part, and calling
// the given list of services with the following input arguments: $bizdoc, $sender and $receiver
// are the normal bizdoc processing service inputs (except with the '$' prefix), $document is the
// parsed content part as an IData document , and $schema is the name of the document reference or
// flat file schema used by the parser.
//
// As it provides logging, content parsing, error handling, and document status updates, the
// $services processing services do not need to include any of this common boilerplate code.
//
// If a custom $catch service is specified, it will be called if an error occurs while processing
// the bizdoc.  The $catch service will be passed the current pipeline, along with the following
// additional arguments:
//   $exception - the actual exception object thrown by the $service
//   $exception.message - the error message
//   $exception.class - the exception object's Java class name
//   $exception.stack - the Java call stack at the time the exception occurred
//
// This service is designed to be called directly from a Trading Networks bizdoc processing rule,
// hence the non-dollar-prefixed bizdoc argument.
//
// Additional arbitrary input arguments for $service (or pub.flatFile:convertToValues/
// pub.xml:xmlStringToXMLNode/pub.xml:xmlNodeToDocument via tundra.tn.document:parse) can be
// specified in the $pipeline document. Fully qualified names will be handled correctly, for
// example an argument named 'example/item[0]' will be converted to an IData document named
// 'example' containing a String list named 'item' with it's first value set accordingly.
//
// Supports 'strict' mode processing of bizdocs: if any $strict error classes are set to 'true' and
// the bizdoc contains errors for any of these classes, the bizdoc will not be processed; instead an
// exception will be thrown and handled by the $catch service. For example, if you have enabled
// duplicate document checking on the Trading Networks document type and do not wish to process
// duplicates, set the $strict/Saving error class to 'true' and duplicate documents will not
// be processed and will instead have their user status set to 'ERROR' (when using the standard
// $catch service).
tundra.tn:chain(bizdoc, $services[], $catch, $finally, $pipeline, $service.input, $encoding, $part, $strict);

// Delivers Trading Networks document (bizdoc) content to the given destination URI.
//
// Supports the following delivery protocols / URI schemes:
//   - file:   writes the given content to the file specified by the destination URI.  The
//             following additional options can be provided via the $pipeline document:
//             - $mode: append / write
//   - http:   transmits the given content to the destination URI. The following additional
//             options can be provided via the $pipeline document:
//             - $method: get / put / post / delete / head / trace / options
//             - $headers/*: additional HTTP headers as required
//             - $authority/user: the username to log on to the remote web server with
//             - $authority/password: the password to log on to the remote web server with
//   - https:   refer to http
//   - mailto: sends an email with the given content attached. An example mailto URI is
//             as follows: mailto:bob@example.com?cc=jane@example.com&subject=Example&body=Example&attachment=message.xml
//             The following additional override options can be provided via the $pipeline
//             document:
//             - $attachment: the attached file's name
//             - $from: email address to send the email from
//             - $subject: the subject line text
//             - $body: the main text of the email
//             - $smtp: an SMTP URI specifying the SMTP server to use (for example,
//               smtp://user:password@host:port), defaults to the SMTP server configured
//               in the Integration Server setting watt.server.smtpServer
//
//
// Variable substitution is performed on all variables specified in the $pipeline document,
// and the $destination URI, allowing for dynamic generation of any of these values. Also,
// if $service is specified, it will be called prior to variable substitution and thus can
// be used to populate the pipeline with variables to be used by the substitution.
//
// This service leverages the Tundra service tundra.content:deliver. Therefore, additional
// delivery protocols can be implemented by creating a service named for the URI scheme in
// the Tundra package folder tundra.support.content.deliver.  Services in this folder should
// implement the tundra.support.content.deliver:handler specification.
//
// Supports 'strict' mode processing of bizdocs: if any $strict error classes are set to 'true' and
// the bizdoc contains errors for any of these classes, the bizdoc will not be processed; instead an
// exception will be thrown and handled by the $catch service. For example, if you have enabled
// duplicate document checking on the Trading Networks document type and do not wish to process
// duplicates, set the $strict/Saving error class to 'true' and duplicate documents will not
// be processed and will instead have their user status set to 'ERROR' (when using the standard
// $catch service).
tundra.tn:deliver(bizdoc, $destination, $encoding, $service, $catch, $finally, $pipeline, $part, $strict);

// Derives a new bizdoc from an existing bizdoc, optionally updating the sender and/or
// receiver on the derivative.
//
// If a $service is specified, it will be called as a processing service for the bizdoc. It can
// return the $derivatives list, thereby allowing for the derivatives to be determined at
// runtime.
//
// Each $derivatives rule can specify a filter condition, which can either be a service which
// implements tundra.tn.schema.derivative:filter specification, or an inline conditional
// statement (as supported by Tundra/tundra.condition:evaluate). Note that the input pipeline
// for an inline conditional statement is the same as the input pipeline for a filter service.
//
// Each $derivatives rule can also specify a list of {key, value} pairs, specified in the
// amendments[] IData array, which will be applied to the default bizdoc content part prior
// to the new copy for the derivative being routed to Trading Networks. The keys in amendments[]
// can be fully-qualified (for example, "a/b/c[0]"), and the values can include percent-delimited
// variable substitution strings which will be substituted prior to being inserted in $document.
//
// Supports 'strict' mode processing of bizdocs: if any $strict error classes are set to 'true' and
// the bizdoc contains errors for any of these classes, the bizdoc will not be processed; instead an
// exception will be thrown and handled by the $catch service. For example, if you have enabled
// duplicate document checking on the Trading Networks document type and do not wish to process
// duplicates, set the $strict/Saving error class to 'true' and duplicate documents will not
// be processed and will instead have their user status set to 'ERROR' (when using the standard
// $catch service).
tundra.tn:derive(bizdoc, $service, $catch, $finally, $pipeline, $derivatives[], $part, $encoding, $strict);

// Logs a message to the Trading Networks activity log.
tundra.tn:log($bizdoc, $type, $class, $summary, $message);

// Processes a Trading Networks document by parsing the given document content part, and calling
// the given service with the following input arguments: $bizdoc, $sender and $receiver are the
// normal bizdoc processing service inputs (except with the '$' prefix), $document is the parsed
// content part as an IData document, and $schema is the name of the document reference or flat
// file schema used by the parser.
//
// Refer to the tundra.tn.schema:processor specification as a guide to the inputs and outputs
// required of the processing service.
//
// As it provides logging, content parsing, error handling, and document status updates, the
// $service processing service does not need to include any of this common boilerplate code.
//
// If a custom $catch service is specified, it will be called if an error occurs while processing
// the bizdoc.  The $catch service will be passed the current pipeline, along with the following
// additional arguments:
//   $exception - the actual exception object thrown by the $service
//   $exception.message - the error message
//   $exception.class - the exception object's Java class name
//   $exception.stack - the Java call stack at the time the exception occurred
//
// This service is designed to be called directly from a Trading Networks bizdoc processing rule,
// hence the non-dollar-prefixed bizdoc argument.
//
// Additional arbitrary input arguments for $service (or pub.flatFile:convertToValues/
// pub.xml:xmlStringToXMLNode/pub.xml:xmlNodeToDocument via tundra.tn.document:parse) can be
// specified in the $pipeline document. Fully qualified names will be handled correctly, for
// example an argument named 'example/item[0]' will be converted to an IData document named
// 'example' containing a String list named 'item' with it's first value set accordingly.
//
// Supports 'strict' mode processing of bizdocs: if any $strict error classes are set to 'true' and
// the bizdoc contains errors for any of these classes, the bizdoc will not be processed; instead an
// exception will be thrown and handled by the $catch service. For example, if you have enabled
// duplicate document checking on the Trading Networks document type and do not wish to process
// duplicates, set the $strict/Saving error class to 'true' and duplicate documents will not
// be processed and will instead have their user status set to 'ERROR' (when using the standard
// $catch service).
tundra.tn:process(bizdoc, $service, $catch, $finally, $pipeline, $part, $encoding, $strict);

// Receives arbitrary (XML or flat file) content and routes it to Trading Networks. The content can be specified
// as a string, byte array, java.io.InputStream, or org.w3c.dom.Node object.
//
// This service is intended to be invoked by clients via HTTP or FTP.
tundra.tn:receive(strict, TN_parms);

// Reprocesses the given document in Trading Networks by rematching it against the
// processing rule base and executing the first processing rule that matches.
tundra.tn:reroute(bizdoc);

// Retrieves arbitrary content (XML, flat files, binary) from the given $source URI, and routes it
// to Trading Networks.
//
// Supports the following retrieval protocols / URI schemes:
//   - file:   processes each file matching the given $source URI with the given processing $service.
//             The file component of the URI can include wildcards or globs (such as *.txt or *.j?r)
//             for matching multiple files at once. For example, file:////server:port/directory/*.txt
//             would process all .txt files in the specified directory.
//             To ensure each file processed is not locked or being written to by another process, the
//             file is first moved to a ./archive directory prior to processing.
//
// Additional retrieval protocols can be implemented by creating a service named for the URI scheme in
// the folder Tundra/tundra.support.content.retrieve.  Services in this folder must implement the
// Tundra/tundra.schema.content.retrieve:handler specification.
//
// Use the $limit input to configure the maximum number of content matches to be processed in a single
// execution (defaults to 1000).
tundra.tn:retrieve($source, $limit, TN_parms);

// One-to-many conversion of an XML or flat file Trading Networks document (bizdoc) to another format.
// Calls the given splitting service, passing the parsed content as an input, and routing the split
// content back to Trading Networks as new documents automatically.
//
// The splitting service must accept a single IData document and return an IData document list, and
// optionally TN_parms. Refer to the tundra.tn.schema:splitter specification as a guide to the inputs
// and outputs required of the splitting service.
//
// Supports 'strict' mode processing of bizdocs: if any $strict error classes are set to 'true' and
// the bizdoc contains errors for any of these classes, the bizdoc will not be processed; instead an
// exception will be thrown and handled by the $catch service. For example, if you have enabled
// duplicate document checking on the Trading Networks document type and do not wish to process
// duplicates, set the $strict/Saving error class to 'true' and duplicate documents will not
// be processed and will instead have their user status set to 'ERROR' (when using the standard
// $catch service).
tundra.tn:split(bizdoc, $service, $catch, $finally, $pipeline, $schema.input, $schema.output, $service.input, $service.output, $encoding.input, $encoding.output, $part, $strict);

// One-to-one conversion of an XML or flat file Trading Networks document (bizdoc) to another format.
// Calls the given translation service, passing the parsed content as an input, and routing the
// translated content back to Trading Networks as a new document automatically.
//
// The translation service must accept a single IData document and return a single IData document,
// and optionally TN_parms. Refer to the tundra.tn.schema:translator specification as a guide to
// the inputs and outputs required of the translation service.
//
// Supports 'strict' mode processing of bizdocs: if any $strict error classes are set to 'true' and
// the bizdoc contains errors for any of these classes, the bizdoc will not be processed; instead an
// exception will be thrown and handled by the $catch service. For example, if you have enabled
// duplicate document checking on the Trading Networks document type and do not wish to process
// duplicates, set the $strict/Saving error class to 'true' and duplicate documents will not
// be processed and will instead have their user status set to 'ERROR' (when using the standard
// $catch service).
tundra.tn:translate(bizdoc, $service, $catch, $finally, $pipeline, $schema.input, $schema.output, $service.input, $service.output, $encoding.input, $encoding.output, $part, $strict);
```

### Content

```java
// Routes arbitrary content specified as a string, byte array, input stream, or IData document to Trading Networks.
//
// Correctly supports large documents, so any document considered large will be routed as a large document in TN,
// unlike the WmTN/wm.tn.doc.xml:routeXML service.
//
// The following values in TN_parms, if specified, will overwrite the normal bizdoc recognised values, allowing
// for sender, receiver, document ID, group ID, conversation ID, and document type to be forced to the specified
// value (even for XML document types):
//   TN_parms/SenderID
//   TN_parms/ReceiverID
//   TN_parms/DocumentID
//   TN_parms/DoctypeID
//   TN_parms/DoctypeName
//   TN_parms/GroupID
//   TN_parms/ConversationID
tundra.tn.content:route($content, $schema, TN_parms);
```

### Document

Bizdoc-related services:

```java
// Trading Networks date attribute transformer which parses the given Trading Networks document (bizdoc)
// attribute value/s with the given datetime pattern (arg) into java.util.Date object/s.
//
// Since the built-in TN date attribute parsing only supports java.text.SimpleDateFormat (refer:
// <http://docs.oracle.com/javase/6/docs/api/java/text/SimpleDateFormat.html>), it is unable to parse
// ISO8601/XML dates, times, and datetimes which may include a timezone specified in the format
// (+|-)HH:mm. As this service supports named patterns for ISO8601/XML and java.text.SimpleDateFormat
// patterns, it can be used instead to parse all date, time and datetime strings.
tundra.attribute.datetime.transformer:parse(values[], arg);

// Trading Networks string transformer which returns whether the given Trading Networks document (bizdoc)
// attribute value/s match the given regular expression pattern (arg).
//
// Refer to <http://docs.oracle.com/javase/6/docs/api/java/util/regex/Pattern.html> for more information
// on regular expression use in Java.
tundra.tn.document.attribute.string.transformer:match(values[], arg);

// Trading Networks string transformer which returns the Trading Networks My Enterprise profile's
// internal ID. This transformer can be used to force the sender or receiver of a document to
// always be the My Enterprise profile.
tundra.tn.document.attribute.string.transformer.profile:self(values[]);

// Trading Networks string transformer which URI decodes the given Trading Networks document (bizdoc)
// attribute value/s.
tundra.tn.document.attribute.string.transformer.uri:decode(values[]);

// Trading Networks string transformer which URI encodes the given Trading Networks document (bizdoc)
// attribute value/s.
tundra.tn.document.attribute.string.transformer.uri:encode(values[]);

// Adds a content part with the given name and content, specified as a string, bytes or stream,
// to the given Trading Networks document (bizdoc).
tundra.tn.document.content:add($bizdoc, $part, $content, $content.type);

// Returns true if the content part identified by the given $part name exists for the given bizdoc.
tundra.tn.document.content:exists($bizdoc, $part);

// Returns true if the given $bizdoc is related to a derived bizdoc with the given $sender
// and $receiver.
tundra.tn.document.derivative:exists($bizdoc, $sender, $receiver);

// Returns the document's content associated with the given part name as a stream. If the part
// name is not provided, the default content part is returned (xmldata for XML; ffdata for Flat
// Files).
tundra.tn.document.content:get($bizdoc, $part, $encoding);

// Derives a new bizdoc from an existing bizdoc, optionally updating the sender and/or
// receiver on the derivative.
//
// An optional list of {key, value} pairs, specified in the $amendments[] IData array,
// will be applied to the default bizdoc content part prior to the new copy for the
// derivative being routed to Trading Networks. The keys in $amendments[] can be fully-
// qualified (for example, "a/b/c[0]"), and the values can include percent-delimited
// variable substitution strings which will be substituted prior to being inserted in
// $document.
tundra.tn.document:derive($bizdoc, $sender, $receiver);

// Returns true if any errors (of the given class, if specified) exist on the given bizdoc.
tundra.tn.document.error:exists($bizdoc, $class);

// Returns the document associated with the given internal ID, optionally
// including the document's content parts.
tundra.tn.document:get($id, $content?);

// Parses the Trading Networks document content part associated with the given part
// name, or the default part if not provided, using the parsing schema configured on
// the document type.
tundra.tn.document:parse($bizdoc, $part, $encoding);

// Relates two Trading Networks documents (bizdocs) together.
tundra.tn.document:relate($bizdoc.source, $bizdoc.target, $relationship);

// Returns the parsing schema associated with the given Trading Networks document.
tundra.tn.document.schema:get($bizdoc);

// Sets user status on the given Trading Networks document.
tundra.tn.document.status:set($bizdoc, $status);

// Returns the Trading Networks document type associated with the given ID or name
// as an IData document.
//
// Use this service in preference to WmTN/wm.tn.doctype:view, as the WmTN service
// returns an object of type com.wm.app.tn.doc.BizDocType which, despite looking
// like one, is not a normal IData document and therefore causes problems in
// Flow services. For example, you cannot branch on fields in the faux document.
tundra.tn.document.type:get($id, $name);

// Returns the parsing schema associated with the given Trading Networks document type.
tundra.tn.document.type.schema:get($type);
```

### Exception

Exception-related services:

```java
// Handles a Trading Networks document processing error by logging the error against
// the document in the activity log, and setting the user status to 'ERROR'.
tundra.tn.exception:handle($bizdoc);
```

### Profile

Partner profile-related services:

```java
// Returns the named delivery method for the given Trading Networks profile.
tundra.tn.profile.delivery:get($profile, $method);

// Returns the Trading Networks profile associated with the given ID. If $type is
// null, then $id must be the internal partner ID, otherwise $type is the external
// ID name to use to find the profile.
tundra.tn.profile:get($id, $type);

// Returns the Trading Networks Enterprise partner profile.
tundra.tn.profile:self();
```

### Queue

Queue processing service versions of the tundra.tn:* meta processing services:

```java
// For each item in the Trading Networks queue, process it with tundra.tn:chain.
tundra.tn.queue:chain(queue, $services[], $catch, $finally, $pipeline, $service.input, $part, $encoding);

// For each item in the Trading Networks queue, process it with tundra.tn:deliver.
tundra.tn.queue:deliver(queue, $destination, $encoding, $service, $catch, $finally, $pipeline, $part);

// For each item in the Trading Networks queue, process it with tundra.tn:derive.
tundra.tn.queue:derive(queue, $service, $catch, $finally, $pipeline, $derivatives, $part, $encoding);

// For each item in the Trading Networks queue, runs the given $service, which must
// implement the bizdoc processing service signature wm.tn.rec:ProcessingService, to
// process the item.
//
// As the above implies, this service lets you use any normal bizdoc processing service
// to process items in a Trading Networks delivery queue.
tundra.tn.queue:each(queue, $service, $pipeline);

// For each item in the Trading Networks queue, process it with tundra.tn:process.
tundra.tn.queue:process(queue, $service, $catch, $finally, $pipeline, $service.input, $part, $encoding);

// For each item in the Trading Networks queue, process it with tundra.tn:reroute.
tundra.tn.queue:reroute(queue);

// For each item in the Trading Networks queue, process it with tundra.tn:split.
tundra.tn.queue:split(queue, $service, $catch, $finally, $pipeline, $schema.input, $schema.output, $service.input, $service.output, $encoding.input, $encoding.output, $part);

// For each item in the Trading Networks queue, process it with tundra.tn:translate.
tundra.tn.queue:translate(queue, $service, $catch, $finally, $pipeline, $schema.input, $schema.output, $service.input, $service.output, , $encoding.input, $encoding.output, $part);
```

### Reliable

Reliable processing services (service execution task) versions of the tundra.tn:*
meta processing services:

```java
// Reliably processes (as a service execution task) a Trading Networks document via tundra.tn:amend.
tundra.tn.reliable:amend(bizdoc, $amendments[], $catch, $finally, $schema, $part.input, $part.output, $encoding.input, $encoding.output, $strict);

// Reliably processes (as a service execution task) a Trading Networks document via
// tundra.tn:chain.
tundra.tn.reliable:chain(bizdoc, $services[], $catch, $finally, $pipeline, $service.input, $part, $encoding);

// Reliably processes (as a service execution task) a Trading Networks document via
// tundra.tn:deliver.
tundra.tn.reliable:deliver(bizdoc, $destination, $encoding, $service, $catch, $finally, $pipeline, $part);

// Reliably processes (as a service execution task) a Trading Networks document via
// tundra.tn:derive.
tundra.tn.reliable:derive(bizdoc, $service, $catch, $finally, $pipeline, $derivatives, $part, $encoding);

// Reliably processes (as a service execution task) a Trading Networks document via
// tundra.tn:process.
tundra.tn.reliable:process(bizdoc, $service, $catch, $finally, $pipeline, $service.input, $part, $encoding);

// Reliably processes (as a service execution task) a Trading Networks document via
// tundra.tn:reroute.
tundra.tn.reliable:reroute(bizdoc);

// Reliably processes (as a service execution task) a Trading Networks document via
// tundra.tn:split.
tundra.tn.reliable:split(bizdoc, $service, $catch, $finally, $pipeline, $schema.input, $schema.output, $service.input, $service.output, $encoding.input, $encoding.output, $part);

// Reliably processes (as a service execution task) a Trading Networks document via
// tundra.tn:translate.
tundra.tn.reliable:translate(bizdoc, $service, $catch, $finally, $pipeline, $schema.input, $schema.output, $service.input, $service.output, $encoding.input, $encoding.output, $part);
```

### Schema

Document schemas and interface specifications:

```java
// An improved version of the WmTN/wm.tn.rec:StringAttributeTransformService specification with
// type constraints provided for the input and output arguments.
tundra.tn.schema.attribute.string.transformer:specification;

// This schema describes the structure for derivative rules used by tundra.tn:derive.
tundra.tn.schema:derivative;

// A compatible superset of wm.tn.rec:ProfileSummary and wm.tn.rec:Profile, with some developer-
// friendly formats for all the external IDs, extended fields, and delivery methods.
tundra.tn.schema:profile;

// Filter services used by tundra.tn:derive must implement this specification. The filter service
// is allowed to edit the $derivative rule, so that it may disable the rule by setting
// $derivative/enabled? to 'false', or specify a different sender or receiver.
tundra.tn.schema.derivative:filter;

// Processing services called by tundra.tn:process can implement this specification.
//
// Inputs:
//   - $document is the parsed bizdoc content for processing. This is the default name
//     for this input parameter. The actual name of the parameter can be changed using
//     the tundra.tn:process $service.input parameter.
//
//   - $schema is the name of the Integration Server document reference or flat file
//     schema used to parse the content into an IData structure.
tundra.tn.schema:processor;

// Splitting services used by tundra.tn:split can implement this specification.
//
// Inputs:
//   - $document is the parsed bizdoc content for splitting. This is the default name
//     for this input parameter. The actual name of the parameter can be changed using
//     tundra.tn:split's $service.input parameter, which allows the use of tundra.tn:split
//     with existing mapping services.
//
//   - $schema is the name of the Integration Server document reference or flat file schema
//     used to parse the content into an IData structure.
//
// Outputs:
//   - $documents[] is the split list of content with which each item of the list will be routed
//     back to Trading Networks as individual new documents. This is the default name for
//     this output parameter. The actual name of the parameter can be changed using the
//     tundra.tn:split's $service.output parameter, which allows the use of tundra.tn:split
//     with existing mapping services.
//
//   - $schemas[] is the list of Integration Server document references or flat file schemas
//     that each $documents[] item conforms to. The length of $schemas[] must match the
//     length of $documents[], and $schema[n] is used to serialize $document[n] to an input
//     stream for routing to Trading Networks.
//
//   - TN_parms provides routing hints for Trading Networks. It can be specified as either a
//     singleton IData or an IData list. If specified as a singleton, it will be used when
//     routing every item in the $documents[] list. If specified as a list, the length of
//     TN_parms[] must match the length of $documents[], and TN_parms[n] will be used when
//     routing $documents[n] to Trading Networks.
tundra.tn.schema:splitter;

// Translation services used by tundra.tn:translate can implement this specification.
//
// Inputs:
//   - $document is the parsed bizdoc content for translation. This is the default name
//     for this input parameter. The actual name of the parameter can be changed using
//     tundra.tn:translate's $service.input parameter, which allows the use of
//     tundra.tn:translate with existing mapping services.
//
//   - $schema is the name of the Integration Server document reference or flat file
//     schema used to parse the content into an IData structure.
//
// Outputs:
//   - $translation is the translated content which will be routed back to Trading Networks
//     as a new document. This is the default name for this output parameter. The actual
//     name of the parameter can be changed using the tundra.tn:translate's $service.output
//     parameter, which allows the use of tundra.tn:translate with existing mapping services.
//
//   - TN_parms provides routing hints for routing the $translation document back to
//     Trading Networks.
tundra.tn.schema:translator;
```

## Contributions

1. Check out the latest master to make sure the feature hasn't been implemented
   or the bug hasn't been fixed yet
2. Check out the issue tracker to make sure someone already hasn't requested it
   and/or contributed it
3. Fork the project
4. Start a feature/bugfix branch
5. Commit and push until you are happy with your contribution
6. Make sure to add tests for it. This is important so it won't break in a future
   version unintentionally

Please try not to mess with the package version, or history. If you want your
own version please isolate it to its own commit, so it can be cherry-picked
around.

## Copyright

Copyright © 2012 Lachlan Dowding. See license.txt for further details.
