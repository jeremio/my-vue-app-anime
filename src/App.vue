<script setup lang="ts">
import { ref, onMounted, onUnmounted, useTemplateRef } from 'vue';
import { animate, createScope, createSpring, createDraggable } from 'animejs';
import vueLogo from './assets/vue.svg';

// État pour compter les rotations
const rotations = ref(0);

// Utilisation de useTemplateRef pour la référence au conteneur racine
const rootEl = useTemplateRef<HTMLElement>('rootContainer');

// Variable pour stocker le scope d'animation (pas besoin d'être réactive)
let animeScope: any = null;

// Initialiser anime.js lorsque le composant est monté
onMounted(() => {
  if (!rootEl.value) return;

  animeScope = createScope({ root: rootEl.value }).add(scope => {
    // Animation de rebond en boucle
    animate('.logo', {
      scale: [
        { to: 1.25, ease: 'inOut(3)', duration: 200 },
        { to: 1, ease: createSpring({ stiffness: 300 }) }
      ],
      loop: true,
      loopDelay: 250,
    });

    // Rendre le logo déplaçable
    createDraggable('.logo', {
      container: [0, 0, 0, 0],
      releaseEase: createSpring({ stiffness: 200 })
    });

    // Méthode pour faire tourner le logo
    scope.add('rotateLogo', (i: number) => {
      animate('.logo', {
        rotate: i * 360,
        ease: 'out(4)',
        duration: 1500,
      });
    });
  });
});

// Nettoyer les instances anime.js lorsque le composant est démonté
onUnmounted(() => {
  if (animeScope) {
    animeScope.revert();
  }
});

// Fonction pour faire tourner le logo au clic
const handleClick = () => {
  if (!animeScope) return;

  rotations.value += 1;
  animeScope.methods.rotateLogo(rotations.value);
};
</script>

<template>
  <div ref="rootContainer">
    <div class="large centered row">
      <img :src="vueLogo" class="logo vue" alt="Vue logo" />
    </div>
    <div class="medium row">
      <fieldset class="controls">
        <button @click="handleClick">rotations: {{ rotations }}</button>
      </fieldset>
    </div>
  </div>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
}

.large.centered.row {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 2rem;
}

.medium.row {
  display: flex;
  justify-content: center;
  margin: 2rem;
}

.controls {
  padding: 1rem;
  border-radius: 8px;
}
</style>