Element Creation Helper
=======================

<xmp class=idl>
interface El {
	constructor(DOMString elementName, optional AttrMap attributes, ...any children);

	/* individual element shorthands */
	static a(optional AttrMap attributes, ...any children);
	static div(optional AttrMap attributes, ...any children);
	...
}

typedef record<DOMString, any>? AttrMap;
</xmp>

El(elementName, attributes, ...children) creates an element with the given name,
attribues,
and children,
as defined below.

Element Names and Namespaces
----------------------------

Unless otherwise specified,
the name must be an HTML, SVG, or MathML element name,
or a custom element name,
or else the function throws.
If a name exists in both HTML and another syntax,
it defaults to the HTML namespace.

The namespace of an element, ambiguous or not,
can be controlled with an "html:", "svg:", or "math:" prefix on the name,
such as "svg:a",
to opt it into a desired namespace;
all of these can be used on non-ambiguous names as well
(but a mismatch, such as "svg:div", throws).

To create elements from any namespace,
you can pass an "xmlns" member in the {{attributes}};
if this isn't the HTML, SVG, or MathML namespace,
then the name is merely checked for overall XML validity,
and is otherwise unconstrained in value.

If a ns prefix is provided on the name *and* an "xmlns" member is provided in the attributes,
it throws.


Attributes, Properties, and Event Listeners
-------------------------------------------

Element attributes, DOM properties, and event listeners
can all be provided upon creation via the AttrMap argument.
The element is first created with the name given by the first argument,
then the attributes are iterated
(this ordering is potentially observable).

For each key, in order:

* If the value is `null` or `undefined`,
	continue to the next key.

	(This allows adding an attribute/etc conditionally.)

* If a key contains *only* lowercase ASCII letters,
	or `"_"` or `"-"`,
	it's treated as an attribute.

	<dl class=switch>
		: If the value is `true`,
		:: Set the named attribute to the empty string.
		: If the value is `false`,
		:: Continue. (Don't set the attribute.)
		: If the key starts with "on" and the value is a function
		:: Add the value as an event listener.
		: If the key starts with "on" and the value is an object with a 'handleEvent' property
		:: Add the value as an event listener. If the object has an 'options' property, pass its value as EventListenerOptions.
		: Otherwise,
		:: set the named attribute to the stringification of the value.
	</dl>

* If a key starts with a `"."`,
	it's treated as a DOM property.

	The named DOM property is set to the value.

Children
--------

First, fully flatten the {{children}} list.

Then, for each |item| of {{children}}:

* If |item| is a Node,
	append it to the element.
* Otherwise,
	create a new Text node
	whose value is the stringification of |item|
	and append it to the element.

Example Implementation
----------------------

<xmp highlight=js>
export function iter(obj) {
	if (!obj) return [];
	var it = obj[Symbol.iterator];
	if (it) return it;
	return Object.entries(obj);
}

export function El(tagname, attrs, ...children) {

	// Handle the built-in ns prefixes
	let ns;
	let softNs;
	if(tagname.startsWith("html:")) {
		ns = "http://www.w3.org/1999/xhtml";
		tagname = tagname.slice(5);
	} else if(tagname.startsWith("svg:")) {
		ns = "http://www.w3.org/2000/svg";
		tagname = tagname.slice(4);
	} else if(tagname.startsWith("math:")) {
		ns = "http://www.w3.org/1998/Math/MathML";
		tagname = tagname.slice(5);
	} else if(htmlTagNames.exec(tagname)) {
		softNs = "http://www.w3.org/1999/xhtml";
	} else if(svgTagNames.exec(tagname)) {
		softNs = "http://www.w3.org/2000/svg";
	} // else match against math tagnames too

	let is;
	const meta = [];
	for (const [k, v] of iter(attrs)) {
		if(k == "xmlns") {
			if(ns !== undefined) throw new DOMException("Can't set namespace both via prefix and explicit ns argument.");
			ns = v;
		} else if(k == "is") {
			is = v;
		} else if(k.startsWith("On")) {
			meta.push(["event", k.slice(3), v]);
		} else if (k[0] == ".") {
			meta.push(["prop", k.slice(1), v]);
		} else {
			meta.push(["attr", k, v]);
		}
	}

	let el;
	if(ns || softNs) {
		el = document.createElementNS(ns || softNs, el, {is});
	} else {
		el = document.createElement(el, {is});
	}

	for(const [type, ...data] of meta) {
		if(type == "attr") {
			el.setAttribute(data[0], data[1]);
		} else if(type == "prop") {
			el[data[0]] = data[1];
		} else if(type == "event") {
			el.addEventListener(data[0], data[1]);
		}
	}

	append(el, children);
	return el;
}


[
	"a",
	"abbr",
	"acronym",
	"address",
	"applet",
	"area",
	"article",
	"aside",
	"audio",
	"b",
	"base",
	"basefont",
	"bdo",
	"big",
	"blockquote",
	"body",
	"br",
	"button",
	"canvas",
	"caption",
	"center",
	"cite",
	"code",
	"col",
	"colgroup",
	"datalist",
	"dd",
	"del",
	"details",
	"dfn",
	"dialog",
	"div",
	"dl",
	"dt",
	"em",
	"embed",
	"fieldset",
	"figcaption",
	"figure",
	"font",
	"footer",
	"form",
	"frame",
	"frameset",
	"head",
	"header",
	"h1",
	"h2",
	"h3",
	"h4",
	"h5",
	"h6",
	"hr",
	"html",
	"i",
	"iframe",
	"img",
	"input",
	"ins",
	"kbd",
	"label",
	"legend",
	"li",
	"link",
	"main",
	"map",
	"mark",
	"meta",
	"meter",
	"nav",
	"nobr",
	"noscript",
	"object",
	"ol",
	"optgroup",
	"option",
	"output",
	"p",
	"param",
	"pre",
	"progress",
	"q",
	"s",
	"samp",
	"script",
	"section",
	"select",
	"small",
	"source",
	"span",
	"strike",
	"strong",
	"style",
	"sub",
	"summary",
	"sup",
	"table",
	"tbody",
	"td",
	"textarea",
	"tfoot",
	"th",
	"thead",
	"time",
	"title",
	"tr",
	"u",
	"ul",
	"var",
	"video",
	"wbr",
	"xmp",
].forEach(tagname => {
	El[tagname] = (...args) => El("html:"+tagname, ...args);
});
[
	"animate",
	"animateMotion",
	"animateTransform",
	"circle",
	"clipPath",
	"defs",
	"desc",
	"discard",
	"ellipse",
	"feBlend",
	"feColorMatrix",
	"feComponentTransfer",
	"feComposite",
	"feConvolveMatrix",
	"feDiffuseLighting",
	"feDisplacementMap",
	"feDistantLight",
	"feDropShadow",
	"feFlood",
	"feFuncA",
	"feFuncB",
	"feFuncG",
	"feFuncR",
	"feGaussianBlur",
	"feImage",
	"feMerge",
	"feMergeNode",
	"feMorphology",
	"feOffset",
	"fePointLight",
	"feSpecularLighting",
	"feSpotLight",
	"feTile",
	"feTurbulence",
	"filter",
	"foreignObject",
	"g",
	"image",
	"line",
	"linearGradient",
	"marker",
	"mask",
	"metadata",
	"mpath",
	"path",
	"pattern",
	"polygon",
	"polyline",
	"radialGradient",
	"rect",
	"set",
	"stop",
	"svg",
	"switch",
	"symbol",
	"text",
	"textPath",
	"tspan",
	"use",
	"view",
].forEach(tagname => {
	El[tagname] = (...args) => El("svg:"+tagname, ...args);
});

function append(el, ...children) {
	const appendChild = (container, child) => {
		if (container instanceof Node) container.appendChild(child);
		else container.push(child);
	};
	for (const child of children) {
		if (child instanceof Node) {
			el.appendChild(child);
		} else if (Array.isArray(child)) {
			append(el, ...child);
		} else {
			el.appendChild(new Text(child));
		}
	}
	return el;
}


var htmlTagNames = /a|abbr|acronym|address|applet|area|article|aside|audio|b|base|basefont|bdo|big|blockquote|body|br|button|canvas|caption|center|cite|code|col|colgroup|datalist|dd|del|details|dfn|dialog|div|dl|dt|em|embed|fieldset|figcaption|figure|font|footer|form|frame|frameset|head|header|h1|h2|h3|h4|h5|h6|hr|html|i|iframe|img|input|ins|kbd|label|legend|li|link|main|map|mark|meta|meter|nav|nobr|noscript|object|ol|optgroup|option|output|p|param|pre|progress|q|s|samp|script|section|select|small|source|span|strike|strong|style|sub|summary|sup|table|tbody|td|textarea|tfoot|th|thead|time|title|tr|u|ul|var|video|wbr|xmp/;

var svgTagNames = /animate|animateMotion|animateTransform|circle|clipPath|defs|desc|discard|ellipse|feBlend|feColorMatrix|feComponentTransfer|feComposite|feConvolveMatrix|feDiffuseLighting|feDisplacementMap|feDistantLight|feDropShadow|feFlood|feFuncA|feFuncB|feFuncG|feFuncR|feGaussianBlur|feImage|feMerge|feMergeNode|feMorphology|feOffset|fePointLight|feSpecularLighting|feSpotLight|feTile|feTurbulence|filter|foreignObject|g|image|line|linearGradient|marker|mask|metadata|mpath|path|pattern|polygon|polyline|radialGradient|rect|set|stop|svg|switch|symbol|text|textPath|tspan|use|view/;