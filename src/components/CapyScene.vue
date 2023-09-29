<script setup lang="ts">
import { watchEffect, shallowRef } from 'vue'
import { Levioso, GLTFModel, useGLTF } from '@tresjs/cientos'
import { TresCanvas, useRenderLoop, type ThreeEvent } from '@tresjs/core'
import { useMouse, useWindowSize } from '@vueuse/core'
import { Vector2, BasicShadowMap } from 'three'
// @ts-expect-error
import fragment from '@/components/shaders/fragment.glsl'
// @ts-expect-error
import vertex from '@/components/shaders/vertex.glsl'

const { x, y } = useMouse()
const { width, height } = useWindowSize()
const box = shallowRef(null)
const spotLightRef = shallowRef(null)

watchEffect(() => {
  x.value = (x.value / width.value) * 2 - 1
  y.value = -(y.value / height.value) * 2 + 1
})

const shader = {
  uniforms: {
    hover: { value: new Vector2(0.5, 0.5) }
  },
  vertexShader: vertex,
  fragmentShader: fragment
}

const updateUniforms = (ev: ThreeEvent<PointerEvent>) => {
  // @ts-expect-error
  ev.object.material.uniforms.hover.value = ev.uv
}

const { onLoop } = useRenderLoop()
const { scene } = await useGLTF(
  'https://raw.githubusercontent.com/gcavanunez/capy-tres/main/public/capybara.gltf',
  {
    draco: true
  }
)

onLoop(({ elapsed }) => {
  if (box.value) {
    // @ts-expect-error
    box.value.map((b, index) => {
      b.value.rotation.y = (index + elapsed) * 0.3
      b.value.rotation.z = (index + elapsed) * 0.3
    })
  }
  if (spotLightRef.value) {
    // @ts-expect-error
    spotLightRef.value.position.set(x.value, y.value, -2)
  }
})
</script>
<template>
  <TresCanvas
    window-size
    receive-shadow
    :shadows="true"
    clear-color="#333"
    :shadowMapType="BasicShadowMap"
    class="over-hidden"
    ref="canvasRef"
  >
    <TresPerspectiveCamera :args="[45, 1, 0.1, 1000]" :position="[0, 0, 1]" />
    <TresMesh :position="[0, 0, -3.5]" receive-shadow>
      <TresPlaneGeometry :args="[10, 10]" />
      <TresMeshStandardMaterial color="#C4C4C4" />
    </TresMesh>
    <!-- Shader wall -->
    <TresMesh @pointer-move="(ev) => updateUniforms(ev)" name="wall">
      <TresPlaneGeometry :args="[2, 1]" />
      <TresShaderMaterial v-bind="shader" :transparent="true" />
    </TresMesh>

    <Suspense>
      <Levioso>
        <primitive
          :object="scene"
          :scale="[0.015, 0.015, 0.015]"
          :position="[0, 0, -3]"
          :rotation-x="Math.PI * 0.5"
          :rotation-z="Math.PI * 0.85"
          :rotation-y="Math.PI * 1"
        />
        <!-- error with compiling  && path="/capybara.gltf" -->
        <!-- <GLTFModel
          :scale="[0.015, 0.015, 0.015]"
          :position="[0, 0, -3]"
          :rotation-x="Math.PI * 0.5"
          :rotation-z="Math.PI * 0.85"
          :rotation-y="Math.PI * 1"
          path="https://raw.githubusercontent.com/gcavanunez/capy-tres/main/public/capybara.gltf"
          draco
          ref="modelRef"
        /> -->
      </Levioso>
    </Suspense>
    <TresPointLight ref="spotLightRef" :args="[0xffffff, 7.5]" cast-shadow name="light" />
  </TresCanvas>
</template>
