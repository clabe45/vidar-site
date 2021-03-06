+++
title = "Custom Effects"
template = "page.html"
+++

<div class="heading-text">Custom Effects</div>

Vidar provides many built-in effects, but you can subclass any of them to create
your own. Note that the term 'effects' refers to 'visual effects' only. Audio
can be manipulated with the [web audio API] and the `audioNode` property of an
[`Audio`] layer, as shown below.

# Simple Visual Effect

```js
class Effect extends vd.effect.Base {
  /**
   * @param {(vd.Movie | vd.layer.Visual)} target - what to apply the effect to
   */
  apply(target) {
    // 'cctx' stands for 'canvas context'; it's the 2d rendering context of the
    // target's `canvas`
    target.cctx.fillStyle = 'blue'
    target.cctx.fillRect(
      0, 0, target.canvas.width / 2, target.canvas.height / 2
    )
  }
}
```

This draws a blue rectangle over the target. `target` can either be a movie or
a visual layer.

# Hardware-Accelerated Pixel Mapping

You can subclass [`vd.effect.Shader`](../api/classes/effect.shader.html) to make
an effect that maps each pixel to a new value using the GPU. The following
example sets each color's red, green and blue channels to `x` divided by the
value of the channel before the effect is applied. The fragment shader source
code is written in GLSL.

```js
class Effect extends vd.effect.Shader {
  constructor(x = 1) {
    super({
      fragmentSource: `
        precision mediump float;

        uniform sampler2D u_Source;  // the original texture (updated every frame)
        uniform float u_X;  // the value of 'x' back in JavaScript (updated every frame)

        varying highp vec2 v_TextureCoord;

        void main() {
          vec4 color = texture2D(u_Source, v_TextureCoord);
          gl_FragColor = vec4(u_X / color.rgb, color.a);
        }
      `,
      uniforms: {
        x: '1f'
      }

      this.x = x
    })
  }
}
```

Note that each uniform is set to the result of calling `val` on the
corresponding property; these properties are dynamic.

# Audio Manipulation

While audio effects are not currently supported, you can modify the audio output
of a layer or movie using the web audio API. Every audio layer has an
`audioNode` property, and the movie has an `actx` ("audio context") property. By
default, each `audioNode` is connected to `actx.destination`, but you can also connect to an
intermediary node that connects to `actx.destination`:

```js
// Disconnect layer from actx.destination
audioLayer.audioNode.disconnect(movie.actx.destination)
// Rewire
audioLayer.audioNode.connect(someAudioNode).connect(movie.actx.destination)
```

[web audio API]: https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API
[`Audio`]: ../api/classes/layer.audio.html
