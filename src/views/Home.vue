<template>
    <div
        ref="content"></div>
    <p style="position: absolute; top: -10px; right: 0;">
        {{ viewport.scale.x }} {{ Math.floor(fps) }}
    </p>
</template>

<script setup>
import { onMounted, ref } from '@vue/runtime-core';
import * as PIXI from 'pixi.js';
import { Viewport } from 'pixi-viewport'
/* ---------------- props ------------------- */
/* ---------------- data -------------------- */
/* ---------------- refs -------------------- */
const content   = ref(null);
const fps       = ref(0);
const viewport  = ref({scale: {x: 0}});


/* ---------------- computed ---------------- */
/* ---------------- functions --------------- */

function addTestStuff(container) {

    // add a red box
    const sprite = container.addChild(new PIXI.Sprite(PIXI.Texture.WHITE))
    sprite.tint = 0xff0000
    sprite.width = sprite.height = 100
    sprite.position.set(100, 100)
    
    for (let i = 0; i < 2; i++) {
        // add a green box
        const sprite = container.addChild(new PIXI.Sprite(PIXI.Texture.WHITE))
        sprite.tint = 0x00ff00
        sprite.width = sprite.height = 100
        sprite.position.set(100+1*i, 100+1*i)
        const basicText = new PIXI.Text('ABCDEFG', {fontFamily: 'Roboto'});
        basicText.x = 120+1*i;
        basicText.y = 150+1*i;
        container.addChild(basicText);
    }
}

function addTextExample(container) {
    const basicText = new PIXI.Text('Marshmallows & almonds', {
        fontFamily: 'Roboto',
        fontSize: 100,
    });
    basicText.x = 50;
    basicText.y = 100;

    container.addChild(basicText);


    const style = new PIXI.TextStyle({
        fontFamily: 'Roboto',
        fontSize: 36,
        fontStyle: 'italic',
        fontWeight: 'bold',
        fill: ['#ffffff', '#00ff99'], // gradient
        stroke: '#4a1850',
        strokeThickness: 5,
        dropShadow: true,
        dropShadowColor: '#000000',
        dropShadowBlur: 4,
        dropShadowAngle: Math.PI / 6,
        dropShadowDistance: 6,
        wordWrap: true,
        wordWrapWidth: 440,
        lineJoin: 'round',
    });

    const richText = new PIXI.Text('Rich text with a lot of options and across multiple lines', style);
    richText.x = 50;
    richText.y = 220;

    container.addChild(richText);

    const skewStyle = new PIXI.TextStyle({
        fontFamily: 'Roboto',
        dropShadow: true,
        dropShadowAlpha: 0.8,
        dropShadowAngle: 2.1,
        dropShadowBlur: 4,
        dropShadowColor: '0x111111',
        dropShadowDistance: 10,
        fill: ['#ffffff'],
        stroke: '#004620',
        fontSize: 60,
        fontWeight: 'lighter',
        lineJoin: 'round',
        strokeThickness: 12,
    });

    const skewText = new PIXI.Text('SKEW IS COOL', skewStyle);
    skewText.skew.set(0.65, -0.3);
    skewText.anchor.set(0.5, 0.5);
    skewText.x = 300;
    skewText.y = 480;

    container.addChild(skewText);

}

function addCanvasWithPixi() {
    // create renderer
    const renderer = new PIXI.Renderer({
        width:      window.innerWidth,
        height:     window.innerHeight,
        backgroundAlpha: 0,
        resolution: window.devicePixelRatio, 
        autoDensity: true,
    });
    content.value.appendChild(renderer.view);

    // create viewport as default container
    const vp = new Viewport({
        interaction: renderer.plugins.interaction // the interaction module is important for wheel to work properly when renderer.view is placed or scaled
    })
    viewport.value = vp;

    // activate plugins
    vp
        .drag({mouseButtons: 'left'})
        .pinch() // touch zoom
        .wheel({smooth: 5})
        .decelerate({friction: 0.89})
    

    vp.on("clicked", (e)=> {
        console.log(e);
    })

    // add stuff --------------------------------
    addTextExample(vp);
    addTestStuff(vp);

    // loop -------------------------------------
    requestAnimationFrame(renderLoop);
    let lastTime = -1;
    let lastScale = vp.scale.x;
    let scaling   = false;
    function renderLoop(time) {
        // fps.value = 1000/(time-lastTime)
        // lastTime = time;

        if (vp.dirty) {
            renderer.render(vp)
            vp.dirty = false

            // detect scaling
            if(vp.scale.x != lastScale) {
                scaling = true;
                lastScale = vp.scale.x;
            }   
        } else if(scaling) {
            // after scale
            scaling = false;
            vp.children.forEach(c => {
                if (c._text) {
                    c.resolution = Math.min(vp.scale.x,4)
                }
            });
            renderer.render(vp)
            viewport.value.scale.x = viewport.value.scale.x;
        }
        requestAnimationFrame(renderLoop);
    }
}

/* ---------------- watchers ---------------- */
onMounted(() => {
    addCanvasWithPixi();
})
</script>

<style>
</style>