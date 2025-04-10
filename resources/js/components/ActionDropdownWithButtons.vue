<template>
  <div>
    <!-- Confirm Action Modal -->
    <component
        v-if="actionModalVisible"
        :show="actionModalVisible"
        class="text-left"
        :is="selectedAction?.component"
        :working="working"
        :selected-resources="selectedResources"
        :resource-name="resourceName"
        :action="selectedAction"
        :errors="errors"
        @confirm="runAction"
        @close="closeConfirmationModal"
    />

    <component
        v-if="responseModalVisible"
        :show="responseModalVisible"
        :is="actionModalReponseData?.component"
        @confirm="handleResponseModalConfirm"
        @close="handleResponseModalClose"
        :data="actionModalReponseData?.payload ?? {}"
    />

    <div class="flex items-center justify-end gap-2">
      <component v-for="action in buttonActions"
                 :key="action.uriKey"
                 :is="Button"
                 :data-action-id="action.uriKey"
                 v-html="action.name"
                 @click.stop="() => handleClick(action)"
                 :title="action.name"
                 :disabled="action.authorizedToRun === false"
      >
      </component>

      <Dropdown>
        <template #default>
          <slot name="trigger">
            <Button
                @click.stop
                :dusk="triggerDuskAttribute"
                variant="ghost"
                icon="ellipsis-horizontal"
                v-tooltip="__('Actions')"
            />
          </slot>
        </template>

        <template #menu>
          <DropdownMenu width="auto">
            <ScrollWrap :height="250">
              <nav
                  class="px-1 divide-y divide-gray-100 dark:divide-gray-800 divide-solid"
              >
                <slot name="menu"/>

                <div v-if="dropdownActions.length > 0">
                  <DropdownMenuHeading v-if="showHeadings">{{
                      __('User Actions')
                    }}
                  </DropdownMenuHeading>

                  <div class="py-1">
                    <DropdownMenuItem
                        v-for="action in dropdownActions"
                        :key="action.uriKey"
                        :data-action-id="action.uriKey"
                        as="button"
                        class="border-none"
                        @click="() => handleClick(action)"
                        :title="action.name"
                        :disabled="action.authorizedToRun === false"
                    >
                      {{ action.name }}
                    </DropdownMenuItem>
                  </div>
                </div>
              </nav>
            </ScrollWrap>
          </DropdownMenu>
        </template>
      </Dropdown>
    </div>
  </div>
</template>

<script setup>
import {Button} from 'laravel-nova-ui'
import {useStore} from 'vuex'
import {useActions} from '@/composables/useActions'
import DropdownMenuHeading from '@/components/Dropdowns/DropdownMenuHeading.vue'
import {computed} from 'vue'

const emitter = defineEmits(['actionExecuted'])

const props = defineProps({
  resource: {},
  resourceName: {},
  viaResource: {},
  viaResourceId: {},
  viaRelationship: {},
  relationshipType: {},
  actions: {type: Array, default: []},
  selectedResources: {type: [Array, String], default: () => []},
  endpoint: {type: String, default: null},
  triggerDuskAttribute: {type: String, default: null},
  showHeadings: {type: Boolean, default: false},
})

const store = useStore()

const {
  errors,
  actionModalVisible,
  responseModalVisible,
  openConfirmationModal,
  closeConfirmationModal,
  closeResponseModal,
  handleActionClick,
  selectedAction,
  working,
  executeAction,
  actionModalReponseData,
} = useActions(props, emitter, store)

const runAction = () => executeAction(() => emitter('actionExecuted'))

const dropdownActions = computed(() => props.actions.filter((action) => !action?.showAsButton))
const buttonActions = computed(() => props.actions.filter((action) => action?.showAsButton && (!action?.hideIfCantRun|| action?.authorizedToRun)))

const handleClick = action => {
  if (action.authorizedToRun !== false) {
    handleActionClick(action.uriKey)
  }
}

const handleResponseModalConfirm = () => {
  closeResponseModal()
  emitter('actionExecuted')
}

const handleResponseModalClose = () => {
  closeResponseModal()
  emitter('actionExecuted')
}
</script>
