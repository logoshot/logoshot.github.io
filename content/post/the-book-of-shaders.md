---
title: "The Book of Shaders"
date: 2022-03-24T12:31:43+08:00
draft: false
---

最近发现的宝藏书籍 [the book of shaders](https://thebookofshaders.com/)

作者制作了一个在线编辑器 [editor.php](https://thebookofshaders.com/edit.php)

# excercises in book

## Algorithmic drwaing

### Colors

#### Compose a gradient that resembles a William Turner sunset
use smoothstep

#### A color has many faces - the relativity of color
``` c
#ifdef GL_ES
precision mediump float;
#endif

uniform vec2 u_resolution;
uniform vec2 u_mouse;
uniform float u_time;

float rect(in vec2 st, in vec2 size){
	size = 0.25-size*0.25;
    vec2 uv = smoothstep(size,size+size*vec2(0.001),st*(1.000-st));
	return uv.x*uv.y;
}

void main() {
    vec2 st = gl_FragCoord.xy/u_resolution.xy;

    vec3 influenced_color = vec3(0.745,0.678,0.539);
    
    vec3 influencing_color_A = vec3(0.653,0.918,0.985); 
    vec3 influencing_color_B = vec3(0.980,0.576,0.113);
    
    vec3 color = mix(influencing_color_A,
                     influencing_color_B,
                     step(.5,st.x));
    float shape=rect(abs((st-vec2(.5,.0))*vec2(2.,1.)),vec2(.05,.125));
    color = mix(color,
               influenced_color,
               shape);

    gl_FragColor = vec4(color,1.0);
}
```
原版代码用非常巧妙的方法画了两个长方形。方法是将横坐标转化为到0.5的距离。然后根据二次函数x*(1-x)的性质，在0.5附近的一小段区间内的值是大于某个值。利用这个性质，可以限定横纵坐标在两个区间内的时候uv两维取值均为1，从而设置这个区间的颜色.

发散一下思维，可以将长方形该成圆.
改动代码如下
```c
float circle(in vec2 st, in float r){
    r*=r;
	return 1.0-smoothstep(r, r+0.001, pow(st.x-0.25,2.0)+pow(st.y-0.5,2.0));
}
float shape=circle(abs((st-vec2(.5,.0))*vec2(1.,1.)),0.1);
```

<img src="https://blog-pictures-2022.oss-accelerate.aliyuncs.com/img/picGo-img/20220324glslcircle.png" style="zoom: 50%" align="center">
