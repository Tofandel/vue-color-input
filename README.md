# vue-color-input
Slick and perfomant vue 3 color picker component whose goal is to replace `<input type=color>`

## Why

### Multi-format
Forget about color conversions: vue-color-input does it for you. Unlike input type=color (which only understands hex) vue-color-input supports all commonly used color models, and by default will output color in the same format that was passed as input. It also has support for alpha channel, unless you specifically disable it. More information below.

### Customizable
Html's native color input is annoying to style. Most likely you'll have to get tricky hiding the original input & binding click event to a presentable-looking div. But it only gets you halfway there cause the color picker popup window is still out of reach and it might look way different in different browsers.
With vue-color-input this poblem is solved. It looks pretty out of the box with the default styles, but it's also intuitive & straight-forward to customize from css. Keep reading for detailed instructions on how.

### Uniform
Not only that native color input looks different in different browsers, it also operates differently, and in some cases it's just not what you expect it to be. Yes, I'm looking at you, Safari. vue-color-input delivers a color picker that looks and performs the same regardless of browser.

### It just works
vue-color-input combines minimalist approach with comprehensive functionality. You *can* customize and extend it to your liking, set it up with additional properties, but you _don't have to_. It works as expected out of the box just as well with only `v-model` provided.

## Usage

### Install
```
npm i vue-color-input
```
### Import
```js
import ColorInput from 'vue-color-input'


app.use(ColorInput)

// OR register component globally

app.component('ColorInput', ColorInput)

// OR locally

export.default {
	components: { ColorInput }
}
```
### Use
```xml
<color-input v-model="color" />
```

# Properties

## v-model

This is where you supply the variable holding the color to be adjusted by end user.
Model value is allowed to be changed externally too, vue-color-input will adjust accordingly.
When first initialized and every time v-model value updates _from outside the component_, incoming color format is stored to be matched by output.

### Input (initial value)

Under the hood vue-color-input uses tinycolor for color conversion. So everything tinycolor [accepts as input](https://github.com/bgrins/TinyColor#accepted-string-input), is valid here as well (both string and object).

### Output (return value)

By default output will be a string or an object in the same color model as the initial value.

For example:
```js
// in parent component

export.default {
	data() {
		color: "rgb(50, 150, 150)"
	}
}
```
```xml
<!-- template -->

<color-input v-model="color" />
```
User adjusts hue to `0`, now `color` becomes
```js
"rgb(150, 50, 50)"
```
Then user adjusts alpha to `0.5`, `color` becomes
```js
"rgba(150, 50, 50, 0.5)"
```

Let's say `color` property was initialy set to be an object:
```js
// in parent component

export.default {
	data() {
		color: { "h": 350, "s": 1, "l": 0.8 }
	}
}
```
In the same scenario the resulting output would be
```js
{ "h": 0, "s": 1, "l": 0.8, "a": 0.5 }
```

vue-color-input will always try to output color in the same color model as the initial value (unless target format is specified explicitly by `format` property.
However in some cases that would not be possible. For those colors it will fall back to different formats.

#### name || hex -> rgba fallback

However, if initial color format was `name` (e.g. `"purple"`) or `hex` (e.g. `"#800080"`), and then alpha is changed to be less than `1`, output will be formatted as `rgba`:
```js
"#cd5c5c" // hex input

/* user changes alpha to 0.9 */

"rgba(205, 92, 92, 0.9)" // rgba output
```

_Note: this behavior does not apply if `format` property is explicitly set to be `hex` or `name`._
_Note 2: if initial color format is `hex8` (e.g. `#800080ff`), output will be `hex8` also, unless specified differently by `format` property._

#### name -> hex fallback

If initial color format was `name`, but the resulting output color does not have a name equivalent, `hex` value will be output instead:
```js
"indianred" // name input

/* user changes hue to 180 */

"#5ccdcc" // hex output
```

#### invalid -> rgb fallback

Invalid color initialy diasplays as black. Default output format will be set to `rgb`:
```js
"ironmanred" // invalid string input

/* user changes alpha to 0.1 */

"rgba(0, 0, 0, 0.1)" // rgb(a) output
```

## format _`optional`_

Here you can supply the color format you want the output to be in.

The value consists of two arguments: format & type. The order of two is inconsequential, e.g. both `"hsl object"` & `"object hsl"` are valid values.
__Format__ is the target color model that the return value is converted to. `[ "rgb", "hsv", "hsl", "hex", "name" ]`
__Type__ is data type of the return value. `[ "string", "object" ]`
If you want to use v-model value for styling, `"string"` type should do the job. On the other hand, if you want to continue processing the data, `"object"` is probably more useful.

Hsv & hsl color component values are presented differently in different output types:
```js
"hsl(0, 53%, 58%)" // "hsl string"

{ "h": 0, "s": 0.531, "l": 0.582, "a": 1 } // "hsl object"
```
Notice how strings contain percent-based values, and object 0-1 floats.

>Note that name & hex formats dont support alpha channel. Specifying either of them as target format will prevent vue-color-input from falling back to rgba. Instead, it will disable alpha slider and always return full opacity color.
>If this is not the behavior that you want, and you'd rather it fall back to rgba to support alpha, you should not specify the format.

#### Type
String

#### Allowed values
```js
// all allowed values
[ "rgb", "rgb object", "rgb string", "hsv", "hsv object", "hsv string", "hsl", "hsl object", "hsl string", "name", "name string", "hex", "hex string", "hex8", "hex8 string" ]
```
_Note: `"name object"`, `"hex object"` & `"hex8 object"`, make no sense and therefore are illegal._
_Note 2: format without type is allowed, type without format is not._

#### Default value
Calculated to match the input.

#### Example
```xml
<color-input v-model="color" format="rgb object" />
```

## position _`optional`_

This is where you specify the position of the popup color picker window relative to the clickable box.

#### Type
String

#### Allowed values
```js
[ "top", "top right", "top left", "top center", "right top", "right", "right bottom", "right center", "bottom right", "bottom", "bottom left", "bottom center", "left top", "left bottom", "left", "left center" ]
```
Pretty intuitive: the first value is the direction from the box in which the popup will appear, the second is how it will align.
_Note: Omitting the second parameter results in center alignment, making `"top"` a shortcut for `"top center"`_

#### Default value
`"bottom"`

#### Example
```xml
<color-input v-model="color" position="right top" />
```

## disabled _`optional`_

Setting this to `true` will make the initial box nonresponsive to user clicks. The popup will not appear.
However the box will still react to v-model changes, should they come from elsewhere.

#### Type
Boolean

#### Allowed values
```js
[ true, false ]
```

#### Default value
`false`

#### Example
```xml
<color-input v-model="color" :disabled="!allowColorAdjustment" />
```

## disable-alpha _`optional`_

If you set this to `true`, alpha slider will be removed from the color picker, and the returned color will always have full opacity.

>Specifying name or hex as the target `format` will make this property default to `true` and ignore any passed value.

#### Type
Boolean

#### Allowed values
```js
[ true, false ]
```

#### Default value
`false`,
`true` if target format is hex or name

#### Example
```xml
<color-input v-model="color" disable-alpha />
```

## disable-text-inputs _`optional`_

With this property you can hide the section of the color picker containing the text inputs.

#### Type
Boolean

#### Allowed values
```js
[ true, false ]
```

#### Default value
`false`

#### Example
```xml
<color-input v-model="color" disable-text-inputs />
```

## transition _`optional`_

Set this to a custom transition name to override factory enter and leave-to transitions of the popup.

This is _not_ the only way to customize color picker transition.
You can also override default transition classes from css.

>More information about Vue enter/leave transitions [here](https://v3.vuejs.org/guide/transitions-enterleave.html).

#### Type
String

#### Default value
`"picker"`

#### Example
```xml
<color-input v-model="color" transition="my-cool-transition" />
```
```css
.my-cool-transition-enter-from,
.my-cool-transition-leave-to {
	transform: rotate(240) scale(.5);
	opacity: 0;
}
.my-cool-transition-enter-active,
.my-cool-transition-leave-active {
	transition: transform .3s, opacity .3s;
}
```



# Styling

As previously mentioned, applying styles to vue-color-input is a breeze. Default CSS is written with custumizability in mind, so anything you want to style will likely work as expected, and the whole component's layout will not get screwed up by that.

To override factory styles, you should address elemets through `.color-input` parent selector, e.g. `.color input .box { }`.

### Class names

| class               | description                                             |
|---------------------|---------------------------------------------------------|
| .color-input        | Root element                                            |
| .box                | Initial clickable box                                   |
| .picker-popup       | Popup color picker window                               |
| .saturation-area    | Picking area where you select saturation and brightness |
| .slider             | Hue and opacity sliders (track)                         |
| .saturation-pointer | Pointer in the saturation-brightness area               |
| .slider-pointer     | Pointer on a slider                                     |
| .text-input         | Text inputs of the color picker                         |

Feel free to scout the HTML for more class names if theese don't cut it.

### DOM structure
```xml
<div class="color-input">
	<div class="box [active] [disabled]"></div>
	<div class="picker-popup"><div>
</div>
```


### Example

```css
.color-input .box {
	/* make clickable box a 100x100 circle */
	width: 100px;
	height: 100px;
	border-radius: 50px;
}
.color-input .picker-popup {
	/* dark mode for popup window */
	background: #000;
	color: #fbfbfb;
	/* and make it wide */
	width: 400px;
}
.color-input .slider {
	/* thin out the sliders and make them wider */
	height: 2px;
	width: 92%;
}
.color-input .saturation-area {
	/* bigger picking area */
	height: 150px;
}
.color-input .slider-pointer {
	/* make slider pointers square-ish and 10x10 */
	border-radius: 4px;
	width: 10px;
	height: 10px;
}
.color-input .saturation-pointer {
	/* increase saturation picker size */
	width: 40px;
	height: 40px;
}
```

# Events

The instance provides hooks for custom handling of key events.

### Event names

| event                | description                                                                                                                                                                         | payload                                                        |
|----------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------|
| mounted              | lifecycle hook, emitted from root component's mounted()                                                                                                                             |                                                                |
| beforeUnmount        | lifecycle hook, emitted from root component's beforeUnmount()                                                                                                                       |                                                                |
| pickStart            | color picking process is initiated, popup is opening                                                                                                                                |                                                                |
| pickEnd              | color picking process is finished, popup will close now                                                                                                                             |                                                                |
| saturationInputStart | saturation-brightness adjustment has begun. This is only emitted when pointerdown inside saturation-brightness area is registered. This will _not_ emit when text inputs are edited | current state of saturation & value (hsv) `{ s: 0.5, v: 0.5 }` |
| saturationInputEnd   | saturation-brightness adjustment has ended. This is only emitted when pointerdown inside saturation-brightness area is registered. This will _not_ emit when text inputs are edited | current state of saturation & value (hsv) `{ s: 0.5, v: 0.5 }` |
| saturationInput      | saturation-brightness is being adjusted. This will emit every time saturation-brightness is changed, including text inputs                                                          | current state of saturation & value (hsv) `{ s: 0.5, v: 0.5 }` |
| hueInputStart        | hue adjustment has begun. This is only emitted when pointerdown over the hue slider is registered. This will _not_ emit when hue is changed from text inputs                        | current state of hue `{ h: 180 }`                              |
| hueInputEnd          | hue adjustment has ended. This is only emitted when pointerdown over the hue slider is registered. This will _not_ emit when hue is changed from text inputs                        | current state of hue `{ h: 180 }`                              |
| hueInput             | hue is being adjusted. This will emit every time hue is changed, including text inputs                                                                                              | current state of hue `{ h: 180 }`                              |
| alphaInputStart      | alpha adjustment has begun. This is only emitted when pointerdown over the alpha slider is registered. This will _not_ emit when alpha is changed from text inputs                  | current state of alpha `{ a: 0.5 }`                            |
| alphaInputEnd        | alpha adjustment has ended. This is only emitted when pointerdown over the alpha slider is registered. This will _not_ emit when alpha is changed from text inputs                  | current state of alpha `{ a: 0.5 }`                            |
| alphaInput           | alpha is being adjusted. This will emit every time alpha is changed, including text inputs                                                                                          | current state of alpha `{ a: 0.5 }`                            |

### Example

```xml
<color-input v-model="color"
@mounted="colorPickerMountedHandler"
@saturationInputStart="saturationStartHandler"
@hueInputStart="hueStartHandler"
@alphaInput="alphaInputHandler" />
```


