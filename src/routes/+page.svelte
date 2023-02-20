<script>
    import * as THREE from 'three';
    import { AsciiEffect } from '$lib/AsciiEffect.js';
    import { onMount } from 'svelte';

    let innerHeight, innerWidth, resizeIntroRenderer = () => {};
    let canvas;
    let frameNum = 340;
    let beforeFillerFrameNum = 340;
    const framesPerViewHeight = 80;
    let scrollY, mx, my;
    $: frame = Math.floor(delay/innerHeight*framesPerViewHeight);
    // video speed control
    let accel = 0.2;
    let delay = 0;
    let introTextures = new Map()
    onMount(() => {
        const preloadImages = cb => {
            let texturesLoaded = 0;
            for (let i = 0; i < frameNum; i++) {
                introTextureLoader.load(currentFrame(i), texture => {
                    introTextures.set(i, texture)
                    texturesLoaded += 1;
                })
            }
            let _, loading = true;
            function loop(){
                _ = requestAnimationFrame(loop)
                console.log('texturesloaded: ' + texturesLoaded)
                if(texturesLoaded == frameNum && loading){
                    loading = false;
                    cb();
                    cancelAnimationFrame(_)
                }
            }
            loop();
        };
        
        let introTextureLoader = new THREE.TextureLoader();
        const introScene = new THREE.Scene();
        const introCamera = new THREE.OrthographicCamera( window.innerWidth / -200, window.innerWidth / 200,
        window.innerHeight / 200, window.innerHeight/-200, 0.01, 100 );
        introCamera.position.z = 1;
        const introRenderer = new THREE.WebGLRenderer({powerPreference: "high-performance" });
        const ascii = new AsciiEffect( introRenderer, ' $@B8&WM#*oaO0QLCJXzcvuxrjft/|;:^\'` ', { invert: false, resolution: 0.2 }); //0.2 is the golden resolution
        introRenderer.setPixelRatio(window.devicePixelRatio);
        introRenderer.setSize( window.innerWidth*1, window.innerHeight*1);
        ascii.setSize( window.innerWidth*1, window.innerHeight*1);

         //resize the intro every time brower changes size
        resizeIntroRenderer = () => {
            if(window.innerWidth / window.innerHeight > 16/9){
                introRenderer.domElement.style.width = (window.innerHeight) * 16 / 9 + 'px';
                introRenderer.domElement.style.height = window.innerHeight + 'px';
                ascii.setSize( window.innerHeight * 16/9, window.innerHeight*1);
                ascii.domElement.style.width = (window.innerHeight) * 16 / 9 + 'px';
                ascii.domElement.style.height = window.innerHeight + 'px';
            } else {
                introRenderer.domElement.style.width = (window.innerWidth)+ 'px';
                introRenderer.domElement.style.height = window.innerWidth * 9 /16 + 'px';
                ascii.setSize( window.innerWidth, window.innerWidth * 9 /16);
                ascii.domElement.style.width = (window.innerWidth)+ 'px';
                ascii.domElement.style.height = window.innerWidth * 9 /16 + 'px';
            }
        }
        resizeIntroRenderer();

        introRenderer.domElement.style.margin = 'auto';
        introRenderer.domElement.style.top = 0;
        introRenderer.domElement.style.bottom = 0;
        introRenderer.domElement.style.position = 'absolute';
        introRenderer.domElement.style.left = 0;
        introRenderer.domElement.style.right = 0;

        let introPlane = new THREE.PlaneGeometry(window.innerWidth/200 * 2, window.innerHeight/200 * 2, 1, 1);
        
        let introPlaneMaterial = new THREE.MeshLambertMaterial( 
            {
                side: THREE.DoubleSide,
                emissive: 0xffffff,
                emissiveMap: introTextureLoader.load(currentFrame(0)),
                emissiveIntensity: 1
            } 
        );
        canvas.appendChild(introRenderer.domElement)
        let introMesh = new THREE.Mesh(introPlane, introPlaneMaterial);
        introMesh.position.set(0,0,0);
        introScene.add(introMesh);
        introRenderer.render(introScene, introCamera)

        ascii.domElement.style.margin = 'auto';
        ascii.domElement.style.top = 0;
        ascii.domElement.style.bottom = 0;
        ascii.domElement.style.position = 'absolute';
        ascii.domElement.style.left = 0;
        ascii.domElement.style.right = 0;
        ascii.domElement.style.color = 'white';
        ascii.domElement.style.background = 'black';
        ascii.domElement.style.transition = 'opacity 1s ease'
        
        canvas.appendChild(ascii.domElement);

        const mainLoop = () => {
            requestAnimationFrame(mainLoop)

            delay += (scrollY - delay) * accel;
            console.log(frame)
            if(introPlaneMaterial.emissiveMap)
                introPlaneMaterial.emissiveMap.dispose();  // to prevent memory leak
            introPlaneMaterial.emissiveMap = introTextures.get(frame);
            introRenderer.render(introScene, introCamera);
            ascii.render(introScene, introCamera);
                ascii.domElement.style.clipPath = `circle(150px at ${mx - ascii.domElement.offsetLeft}px ${my - ascii.domElement.parentElement.offsetTop}px)`
        }
        preloadImages(mainLoop);
    })

    const currentFrame = index => {
        return index < beforeFillerFrameNum - 2 ? `/intro/webp/px-Comp 1_${index.toString().padStart(5, '0')}.webp`
        :  `/intro/webp/px-Comp 1_${(beforeFillerFrameNum-2).toString().padStart(5, '0')}.webp`
    };
</script>
<svelte:window bind:scrollY bind:innerHeight bind:innerWidth on:resize={resizeIntroRenderer} 
on:mousemove={
    e => {
        mx = e.clientX; 
        my = e.clientY
    }
}></svelte:window>
<svelte:head>
    
    <style>
        *{
            box-sizing: border-box
        }
        body{
            background: #000;
            height: 1000vh;
            padding: 0;
            margin: 0;
        }
    </style>
</svelte:head>

<div bind:this={canvas} class="canvas"></div>

<style>
    .canvas{
        position: fixed;
        top: 0; left: 0;
        height: 100vh;
        width: 100vw;
    }
</style>