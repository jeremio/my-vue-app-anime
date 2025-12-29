<script setup lang="ts">
import { ref, onMounted, onUnmounted, useTemplateRef } from 'vue';
import { animate, createScope, spring, createDraggable } from 'animejs';
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
        { to: 1, ease: spring({ stiffness: 300 }) }
      ],
      loop: true,
      loopDelay: 250,
    });

    // Rendre le logo déplaçable
    createDraggable('.logo', {
      container: [0, 0, 0, 0],
      releaseEase: spring({ stiffness: 200 })
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

/**
 * Hook de cycle de vie qui s'exécute lorsque le composant est démonté.
 * Important pour nettoyer les ressources pour éviter les fuites de mémoire.
 */
onUnmounted(() => {
  if (animeScope) {
    // "revert" arrête et supprime toutes les instances anime.js du scope.
    animeScope.revert();
  }
});

/**
 * Gestionnaire de clic pour le bouton.
 * Incrémente le compteur de rotations et appelle la méthode d'animation.
 */
const handleClick = () => {
  if (!animeScope) return;

  rotations.value += 1;
  // Utilise la méthode enregistrée dans le scope pour déclencher l'animation.
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
  will-change: transform;
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