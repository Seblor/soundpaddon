<script lang="ts">
  import { onMount } from "svelte";
  import { sounds } from "../../../stores/sounds";
  import { socket } from "../../../client/connections";
  import { soundOrder } from "../../../stores/mirror-layout";
  import { get } from "svelte/store";
  import { checkIsDemo } from "$lib/utils/misc";

  socket.on("sounds", (soundsList) => {
    sounds.set(soundsList);
  });

  onMount(() => {
    if (checkIsDemo() === false) {
      socket.connect();
    }

    sounds.subscribe((sounds) => {
      if (sounds.length === 0) return;

      const oldSoundOrder = get(soundOrder);

      // Adding new sounds to the sound order
      sounds.forEach((sound) => {
        if (
          oldSoundOrder.find((soundUrl) => soundUrl === sound.url) === undefined
        ) {
          soundOrder.update((order) => {
            order.push(sound.url);
            return order;
          });
        }
      });

      // Removing sounds that are no longer in the sound list
      oldSoundOrder.forEach((soundUrl) => {
        if (sounds.find((sound) => sound.url === soundUrl) === undefined) {
          soundOrder.update((order) => {
            order.splice(order.indexOf(soundUrl), 1);
            return order;
          });
        }
      });
    });
  });
</script>

<slot />
