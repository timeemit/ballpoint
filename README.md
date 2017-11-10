# Ballpoint

Stop writing your OpenAPI file by hand and compose with __Ballpoint__

> A ballpoint pen was conceived and developed as a cleaner and more reliable alternative to dip pens and fountain pens, and it is now the world's most-used writing instrument: millions are manufactured and sold daily.

From [Wikipedia's Ballpoint Pen](https://en.wikipedia.org/wiki/Ballpoint_pen)

## Install

```
npm install -g ballpoint
```

## Use

Add operations for a new model with an array of properties:

```
ballpoint compose MODEL property1[:type1] [property2[:type2] ...]
```

Property is the name


## Common Options

Specify the swaggerfile content type:

```
--format=[json|yaml]
```

