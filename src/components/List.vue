<script setup lang="ts">
import Task from "./Task.vue";
import { ref, onMounted, onUnmounted } from "vue";
import { monitorForElements } from "@atlaskit/pragmatic-drag-and-drop/element/adapter";
import { extractClosestEdge } from "@atlaskit/pragmatic-drag-and-drop-hitbox/closest-edge";
import { reorderWithEdge } from "@atlaskit/pragmatic-drag-and-drop-hitbox/util/reorder-with-edge";
import { triggerPostMoveFlash } from "@atlaskit/pragmatic-drag-and-drop-flourish/trigger-post-move-flash";
import { isTaskData, getTasks, type TTask } from "./task-data";

const tasks = ref<TTask[]>(getTasks());

let cleanup = () => { }
onMounted(() => {
  cleanup = monitorForElements({
    canMonitor({ source }) {
      return isTaskData(source.data);
    },
    onDrop({ location, source }) {
      const target = location.current.dropTargets[0];
      if (!target) {
        return;
      }

      const sourceData = source.data;
      const targetData = target.data;

      if (!isTaskData(sourceData) || !isTaskData(targetData)) {
        return;
      }

      const indexOfSource = tasks.value.findIndex(
        (task) => task.id === sourceData.taskId
      );
      const indexOfTarget = tasks.value.findIndex(
        (task) => task.id === targetData.taskId
      );

      if (indexOfTarget < 0 || indexOfSource < 0) {
        return;
      }

      const closestEdgeOfTarget = extractClosestEdge(targetData);

      // Using `flushSync` so we can query the DOM straight after this line
      tasks.value = reorderWithEdge({
        list: tasks.value,
        startIndex: indexOfSource,
        indexOfTarget,
        closestEdgeOfTarget,
        axis: "vertical",
      });

      // Being simple and just querying for the task after the drop.
      // We could use react context to register the element in a lookup,
      // and then we could retrieve that element after the drop and use
      // `triggerPostMoveFlash`. But this gets the job done.
      const element = document.querySelector(
        `[data-task-id="${sourceData.taskId}"]`
      );
      if (element instanceof HTMLElement) {
        triggerPostMoveFlash(element);
      }
    },
  });
});

onUnmounted(() => {
  cleanup()
})
</script>

<template>
  <div class="pt-6 my-0 mx-auto w-[420px]">
    <div class="flex flex-col gap-2 border border-solid rounded p-2">
      <Task v-for="task in tasks" :key="task.id" :task="task" />
    </div>
  </div>
</template>
