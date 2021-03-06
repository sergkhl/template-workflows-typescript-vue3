<template>
  <div ref="rootRef" class="drag-container">
    <ul class="drag-list">
      <li
        v-for="(stage, index) in stages"
        :key="stage"
        class="drag-column"
        :class="{ ['drag-column-' + stage]: true }"
      >
        <span class="drag-column-header">
          <slot :name="stage">
            <h2>{{ stage }}</h2>
          </slot>
        </span>
        <div class="drag-options"></div>
        <ul
          :ref="addListRef(index)"
          class="drag-inner-list"
          :data-status="stage"
        >
          <transition-group name="right">
            <li
              v-for="block in getBlocks(stage)"
              :key="block[idProp]"
              class="drag-item"
              :data-block-id="block[idProp]"
            >
              <slot :name="block[idProp]">
                <strong>{{ block[statusProp] }}</strong>
                <div>{{ block[idProp] }}</div>
              </slot>
            </li>
          </transition-group>
        </ul>
        <div class="drag-column-footer">
          <slot :name="`footer-${stage}`"></slot>
        </div>
      </li>
    </ul>
  </div>
</template>

<script lang="ts">
import { useKanban } from '@/core/hooks/useKanban'
import { IBlock } from '@/core/types'
import { DragulaOptions } from 'dragula'
import {
  computed,
  defineComponent,
  onBeforeMount,
  onMounted,
  onUpdated,
  PropType,
  Ref,
  ref,
} from 'vue'
import { AnyStateNodeDefinition } from 'xstate'

// interface IKanbanProps {
//   stages: string[]
//   blocks: IBlock[]
//   config: DragulaOptions
//   stateMachineConfig: AnyStateNodeDefinition
// }

export default defineComponent({
  name: 'Kanban',
  props: {
    stages: {
      type: Array as PropType<string[]>,
      required: true,
    },
    blocks: {
      type: Array as PropType<IBlock[]>,
      required: true,
    },
    config: {
      type: Object as PropType<DragulaOptions>,
      default: () => ({}),
    },
    stateMachineConfig: {
      type: Object as PropType<AnyStateNodeDefinition>,
      default: null,
    },
  },
  emits: [
    'init',
    'cloned',
    'out',
    'over',
    'shadow',
    'remove',
    'cancel',
    'update-block',

    'drag',
    'dragend',
    'drop',
  ],

  setup(props, { emit }) {
    const listRefs = (ref([]) as unknown) as Ref<Element[]>
    const rootRef = (ref(null) as unknown) as Ref<Element>
    const idProp = 'id'
    const statusProp = 'status'
    const { drake, allowedTargets, forbiddenTargets, getBlocks } = useKanban(
      props,
      emit,
      idProp,
      statusProp,
      rootRef,
      listRefs,
    )

    onUpdated(() => {
      if (drake.value && listRefs.value) {
        drake.value.containers = listRefs.value
      }
    })

    const addListRef = (index: number) => (el: Element) => {
      listRefs.value[index] = el
    }

    return { getBlocks, idProp, statusProp, listRefs, rootRef, addListRef }
  },
})
</script>
