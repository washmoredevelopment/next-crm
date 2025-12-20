<template>
  <div v-if="show" class="flex flex-col w-[352px] text-base">
    <!-- ToDo Header -->
    <div class="flex items-center justify-between p-4.5 text-ink-gray-7 text-lg font-medium">
      <div
        class="flex items-center gap-x-2"
        :class="mode == 'edit' && 'cursor-pointer hover:text-ink-gray-8'"
        @click="mode == 'edit' && switchToDetails()"
      >
        <LucideChevronLeft v-if="mode == 'edit'" class="size-4" />
        {{ __(title) }}
      </div>
      <div class="flex items-center gap-x-1">
        <ShortcutTooltip v-if="mode == 'details'" :label="__('Edit todo')" combo="Enter">
          <Button :icon="EditIcon" variant="ghost" @click="switchToEdit" />
        </ShortcutTooltip>
        <ShortcutTooltip
          v-if="mode === 'edit' || mode === 'details'"
          :label="__('Delete todo')"
          combo="Delete"
          :alt-combos="['Backspace']"
        >
          <Button icon="trash-2" variant="ghost" @click="deleteToDo" />
        </ShortcutTooltip>
        <ShortcutTooltip :label="__('Close panel')" combo="Esc">
          <Button icon="x" variant="ghost" @click="close" />
        </ShortcutTooltip>
      </div>
    </div>

    <!-- ToDo Details View -->
    <div v-if="mode == 'details'" class="flex flex-col overflow-y-auto">
      <div class="flex items-start gap-2 px-4.5 py-3 pb-0" @dblclick="switchToEdit">
        <component :is="priorityIcon" class="size-4 mt-1" />
        <div class="flex flex-col gap-[3px]">
          <div class="text-ink-gray-8 font-semibold text-xl">
            {{ _todo.title || __('(No title)') }}
          </div>
          <div class="text-ink-gray-6 text-p-base">{{ formattedDate }}</div>
        </div>
      </div>

      <!-- Status Badge -->
      <div class="px-4.5 py-2">
        <div class="flex items-center gap-2 text-ink-gray-7">
          <ToDoStatusIcon :status="_todo.status" class="size-4" />
          <span>{{ _todo.status }}</span>
        </div>
      </div>

      <!-- Reference Link -->
      <div v-if="_todo.referenceDocname" class="mx-4.5 my-2.5 border-t border-outline-gray-1" />
      <div v-if="_todo.referenceDocname" class="flex items-center px-4.5 py-1 text-ink-gray-7">
        <component :is="referenceIcon" class="size-4" />
        <Link
          class="[&_button]:bg-surface-white [&_button]:select-text [&_button]:text-ink-gray-7 [&_button]:cursor-text"
          v-model="_todo.referenceDocname"
          :doctype="_todo.referenceDoctype"
          :disabled="true"
        />
        <Button variant="ghost" @click="redirectToReference">
          <template #icon>
            <ArrowUpRightIcon class="size-4 text-ink-gray-7" />
          </template>
        </Button>
      </div>

      <!-- Assignee -->
      <div class="mx-4.5 my-2.5 border-t border-outline-gray-1" />
      <div class="flex items-center gap-2 px-4.5 py-2 text-ink-gray-7">
        <UserAvatar :user="_todo.allocatedTo" class="!size-5" />
        <span>{{ assigneeName }}</span>
      </div>

      <!-- Description -->
      <div
        v-if="_todo.description && _todo.description !== '<p></p>'"
        class="mx-4.5 my-2.5 border-t border-outline-gray-1"
      />
      <div v-if="_todo.description && _todo.description !== '<p></p>'">
        <div class="flex gap-2 items-center text-ink-gray-7 px-4.5 py-1">
          <DescriptionIcon class="size-4" />
          {{ __('Description') }}
        </div>
        <div class="px-4.5 py-2 text-ink-gray-7 text-p-base" v-html="_todo.description" />
      </div>
    </div>

    <!-- ToDo Edit View -->
    <div v-else class="flex flex-col overflow-y-auto">
      <!-- Title -->
      <div class="px-4.5 py-3">
        <TextInput
          ref="todoTitle"
          class="w-full"
          variant="outline"
          v-model="_todo.title"
          :debounce="500"
          :placeholder="__('ToDo title')"
        />
      </div>

      <!-- Description -->
      <div class="px-4.5 py-2">
        <div class="mb-1.5 text-xs text-ink-gray-5">{{ __('Description') }}</div>
        <TextEditor
          variant="outline"
          editor-class="!prose-sm overflow-auto min-h-[100px] max-h-40 py-1.5 px-2 rounded border border-outline-gray-modals bg-surface-gray-2"
          :content="_todo.description"
          :fixed-menu="true"
          @change="(val) => (_todo.description = val)"
          :placeholder="__('Add description...')"
        />
      </div>

      <!-- Status -->
      <div class="flex items-center justify-between px-4.5 py-[7px] text-ink-gray-7">
        <div>{{ __('Status') }}</div>
        <Dropdown :options="statusOptions">
          <Button :label="_todo.status" class="justify-between">
            <template #prefix>
              <ToDoStatusIcon :status="_todo.status" class="size-4" />
            </template>
          </Button>
        </Dropdown>
      </div>

      <!-- Priority -->
      <div class="flex items-center justify-between px-4.5 py-[7px] text-ink-gray-7">
        <div>{{ __('Priority') }}</div>
        <Dropdown :options="priorityOptions">
          <Button :label="_todo.priority" class="justify-between">
            <template #prefix>
              <ToDoPriorityIcon :priority="_todo.priority" class="size-4" />
            </template>
          </Button>
        </Dropdown>
      </div>

      <!-- Assignee -->
      <div class="flex items-center justify-between px-4.5 py-[7px] text-ink-gray-7">
        <div>{{ __('Assignee') }}</div>
        <Link
          class="w-[180px]"
          :value="getUser(_todo.allocatedTo).full_name"
          doctype="User"
          @change="(option) => (_todo.allocatedTo = option)"
          :placeholder="__('Select user')"
          :hideMe="true"
          :filters="[['User', 'user_type', '=', 'System User']]"
        >
          <template #prefix>
            <UserAvatar class="mr-2 !h-4 !w-4" :user="_todo.allocatedTo" />
          </template>
          <template #item-prefix="{ option }">
            <UserAvatar class="mr-2" :user="option.value" size="sm" />
          </template>
        </Link>
      </div>

      <!-- Date -->
      <div class="flex items-center justify-between px-4.5 py-[7px] text-ink-gray-7">
        <div>{{ __('Due Date') }}</div>
        <DatePicker
          class="w-[180px]"
          variant="outline"
          v-model="_todo.date"
          :format="'MMM D, YYYY'"
          :placeholder="__('Select date')"
          :clearable="false"
        />
      </div>

      <!-- Reference Linking -->
      <div class="mx-4.5 my-2.5 border-t border-outline-gray-1" />
      <div class="flex items-center justify-between px-4.5 py-[7px] text-ink-gray-7">
        <div>{{ __('Link to') }}</div>
        <Dropdown :options="referenceTypeOptions">
          <Button :label="_todo.referenceDoctype || __('None')" class="justify-between w-[180px]">
            <template #prefix>
              <component :is="referenceIcon" class="size-4" />
            </template>
          </Button>
        </Dropdown>
      </div>
      <div v-if="_todo.referenceDoctype" class="flex items-center justify-between px-4.5 py-[7px] text-ink-gray-7">
        <div>{{ _todo.referenceDoctype }}</div>
        <Link
          class="w-[180px]"
          v-model="_todo.referenceDocname"
          :doctype="_todo.referenceDoctype"
          :placeholder="__('Select {0}', [_todo.referenceDoctype])"
          :filters="referenceFilters"
        />
      </div>

      <!-- Google Calendar Sync -->
      <div class="mx-4.5 my-2.5 border-t border-outline-gray-1" />
      <div class="flex items-center justify-between px-4.5 py-[7px] text-ink-gray-7">
        <div class="flex items-center gap-2">
          <FormControl
            type="checkbox"
            v-model="_linkedEvent.syncWithGoogleCalendar"
            @change="(e) => (_linkedEvent.syncWithGoogleCalendar = e.target.checked)"
          />
          <label
            class="text-sm cursor-pointer"
            @click="_linkedEvent.syncWithGoogleCalendar = !_linkedEvent.syncWithGoogleCalendar"
          >
            {{ __('Sync with Google Calendar') }}
          </label>
        </div>
      </div>
      <div
        v-if="_linkedEvent.syncWithGoogleCalendar"
        class="flex items-center justify-between px-4.5 py-[7px] text-ink-gray-7"
      >
        <div>{{ __('Calendar') }}</div>
        <Link
          class="w-[180px]"
          v-model="_linkedEvent.googleCalendar"
          doctype="Google Calendar"
          :placeholder="__('Select calendar')"
          :filters="{ enable: 1 }"
        />
      </div>
      <div v-if="_linkedEvent.syncWithGoogleCalendar" class="px-4.5 py-2">
        <div class="mb-1.5 text-xs text-ink-gray-5">{{ __('Invite participants (emails)') }}</div>
        <MultiValueInput
          v-model="eventParticipants"
          class="w-full"
          :placeholder="__('Add email addresses')"
          :errorMessage="(value) => __('Invalid email: {0}', [value])"
          :validate="validateEmail"
          :error="(value) => !validateEmail(value)"
          :triggerKeys="['Enter', ',', 'Tab', ' ']"
        />
      </div>

      <!-- Error Message -->
      <ErrorMessage v-if="error" :message="error" class="mx-4.5 my-2" />

      <!-- Save Button -->
      <div class="px-4.5 py-3">
        <Button
          variant="solid"
          :label="isNewTodo ? __('Create') : __('Save')"
          class="w-full"
          :disabled="!canSave"
          @click="saveToDo"
        />
      </div>
    </div>
  </div>
</template>

<script setup>
import ShortcutTooltip from '@/components/ShortcutTooltip.vue'
import ToDoStatusIcon from '@/components/Icons/ToDoStatusIcon.vue'
import ToDoPriorityIcon from '@/components/Icons/ToDoPriorityIcon.vue'
import LeadsIcon from '@/components/Icons/LeadsIcon.vue'
import OpportunitiesIcon from '@/components/Icons/OpportunitiesIcon.vue'
import ProspectsIcon from '@/components/Icons/ProspectsIcon.vue'
import LinkIcon from '@/components/Icons/LinkIcon.vue'
import DescriptionIcon from '@/components/Icons/DescriptionIcon.vue'
import ArrowUpRightIcon from '@/components/Icons/ArrowUpRightIcon.vue'
import EditIcon from '@/components/Icons/EditIcon.vue'
import UserAvatar from '@/components/UserAvatar.vue'
import Link from '@/components/Controls/Link.vue'
import MultiValueInput from '@/components/Controls/MultiValueInput.vue'
import { usersStore } from '@/stores/users'
import { globalStore } from '@/stores/global'
import { todoStatusOptions, todoPriorityOptions } from '@/utils'
import { useKeyboardShortcuts } from '@/composables/useKeyboardShortcuts'
import {
  TextInput,
  TextEditor,
  Dropdown,
  DatePicker,
  FormControl,
  ErrorMessage,
  createDocumentResource,
  call,
  dayjs,
  toast,
} from 'frappe-ui'
import { ref, computed, watch, nextTick } from 'vue'
import { useRouter } from 'vue-router'
import LucideChevronLeft from '~icons/lucide/chevron-left'

const props = defineProps({
  todo: {
    type: Object,
    default: () => ({}),
  },
  mode: {
    type: String,
    default: 'details',
  },
})

const show = defineModel()
const emit = defineEmits(['close', 'save', 'delete', 'edit', 'details'])

const router = useRouter()
const { getUser } = usersStore()
const { $dialog } = globalStore()

const todoTitle = ref(null)
const error = ref(null)

const _todo = ref({
  id: '',
  name: '',
  title: '',
  description: '',
  status: 'Open',
  priority: 'Medium',
  allocatedTo: '',
  date: '',
  referenceDoctype: '',
  referenceDocname: '',
})

const _linkedEvent = ref({
  syncWithGoogleCalendar: false,
  googleCalendar: '',
  name: '',
})

const eventParticipants = ref([])

const isNewTodo = computed(() => !_todo.value.name)

const title = computed(() => {
  if (props.mode === 'new') return __('New ToDo')
  if (props.mode === 'edit') return __('Edit ToDo')
  return __('ToDo')
})

const formattedDate = computed(() => {
  if (!_todo.value.date) return __('No due date')
  return dayjs(_todo.value.date).format('ddd, D MMM YYYY')
})

const assigneeName = computed(() => {
  if (!_todo.value.allocatedTo) return __('Unassigned')
  return getUser(_todo.value.allocatedTo).full_name || _todo.value.allocatedTo
})

const priorityIcon = computed(() => {
  return ToDoPriorityIcon
})

const referenceIcon = computed(() => {
  switch (_todo.value.referenceDoctype) {
    case 'Lead':
      return LeadsIcon
    case 'Opportunity':
      return OpportunitiesIcon
    case 'Prospect':
      return ProspectsIcon
    default:
      return LinkIcon
  }
})

const referenceFilters = computed(() => {
  if (_todo.value.referenceDoctype === 'Lead') {
    return { status: ['!=', 'Converted'] }
  }
  return {}
})

const statusOptions = todoStatusOptions((status) => {
  _todo.value.status = status
})

const priorityOptions = todoPriorityOptions((priority) => {
  _todo.value.priority = priority
})

const referenceTypeOptions = [
  {
    label: __('None'),
    onClick: () => {
      _todo.value.referenceDoctype = ''
      _todo.value.referenceDocname = ''
    },
  },
  {
    label: __('Lead'),
    onClick: () => {
      _todo.value.referenceDoctype = 'Lead'
      _todo.value.referenceDocname = ''
    },
  },
  {
    label: __('Opportunity'),
    onClick: () => {
      _todo.value.referenceDoctype = 'Opportunity'
      _todo.value.referenceDocname = ''
    },
  },
  {
    label: __('Prospect'),
    onClick: () => {
      _todo.value.referenceDoctype = 'Prospect'
      _todo.value.referenceDocname = ''
    },
  },
]

const canSave = computed(() => {
  return _todo.value.title || _todo.value.description
})

// Watch for todo prop changes
watch(
  () => props.todo,
  (newTodo) => {
    if (newTodo?.id) {
      fetchToDo(newTodo)
    }
  },
  { immediate: true },
)

watch(
  () => props.mode,
  (newMode) => {
    if (newMode === 'edit' || newMode === 'new') {
      focusOnTitle()
    }
  },
)

function fetchToDo(todo) {
  if (todo.todoName) {
    // Fetch full todo data
    const todoResource = createDocumentResource({
      doctype: 'ToDo',
      name: todo.todoName,
      fields: ['*'],
      onSuccess: (data) => {
        parseToDo(data)
      },
    })
    todoResource.reload()
  } else {
    // New todo from calendar click
    _todo.value = {
      id: '',
      name: '',
      title: '',
      description: '',
      status: 'Open',
      priority: 'Medium',
      allocatedTo: getUser().name,
      date: todo.fromDate || dayjs().format('YYYY-MM-DD'),
      referenceDoctype: '',
      referenceDocname: '',
    }
  }
}

function parseToDo(data) {
  _todo.value = {
    id: `todo-${data.name}`,
    name: data.name,
    title: data.custom_title || extractTitle(data.description),
    description: data.description || '',
    status: data.status,
    priority: data.priority,
    allocatedTo: data.allocated_to,
    date: data.date,
    referenceDoctype: data.reference_type || '',
    referenceDocname: data.reference_name || '',
  }

  // Check for linked event
  if (data.custom_linked_event) {
    _linkedEvent.value.name = data.custom_linked_event
    // Fetch event details for google calendar sync
    call('frappe.client.get_value', {
      doctype: 'Event',
      filters: { name: data.custom_linked_event },
      fieldname: ['sync_with_google_calendar', 'google_calendar'],
    }).then((res) => {
      if (res?.message) {
        _linkedEvent.value.syncWithGoogleCalendar = res.message.sync_with_google_calendar
        _linkedEvent.value.googleCalendar = res.message.google_calendar
      }
    })
  } else {
    _linkedEvent.value = {
      syncWithGoogleCalendar: getUser().google_calendar ? true : false,
      googleCalendar: getUser().google_calendar || '',
      name: '',
    }
  }
}

function extractTitle(description) {
  if (!description) return ''
  const div = document.createElement('div')
  div.innerHTML = description
  const text = div.textContent || div.innerText || ''
  return text.split('\n')[0].substring(0, 50) || __('ToDo')
}

function focusOnTitle() {
  nextTick(() => {
    todoTitle.value?.el?.focus()
  })
}

function validateEmail(email) {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)
}

function redirectToReference() {
  if (!_todo.value.referenceDocname) return

  const refType = _todo.value.referenceDoctype
  let routeName = refType
  let params = {}

  if (refType === 'Lead') {
    params = { leadId: _todo.value.referenceDocname }
  } else if (refType === 'Opportunity') {
    params = { opportunityId: _todo.value.referenceDocname }
  } else if (refType === 'Prospect') {
    params = { prospectId: _todo.value.referenceDocname }
  } else {
    return
  }

  router.push({ name: routeName, params })
}

function switchToEdit() {
  emit('edit', _todo.value)
}

function switchToDetails() {
  emit('details', _todo.value)
}

async function saveToDo() {
  error.value = null

  if (!_todo.value.title && !_todo.value.description) {
    error.value = __('Title or description is required')
    return
  }

  if (!_todo.value.allocatedTo) {
    _todo.value.allocatedTo = getUser().name
  }

  try {
    // Handle Google Calendar sync
    if (_linkedEvent.value.syncWithGoogleCalendar) {
      if (!_linkedEvent.value.googleCalendar) {
        error.value = __('Please select a Google Calendar')
        return
      }
      await handleGoogleCalendarSync()
    }

    const todoData = {
      custom_title: _todo.value.title,
      description: _todo.value.description,
      status: _todo.value.status,
      priority: _todo.value.priority,
      allocated_to: _todo.value.allocatedTo,
      assigned_by: getUser().name,
      date: _todo.value.date,
      reference_type: _todo.value.referenceDoctype || null,
      reference_name: _todo.value.referenceDocname || null,
      custom_linked_event: _linkedEvent.value.name || null,
    }

    if (_todo.value.name) {
      // Update existing
      await call('frappe.client.set_value', {
        doctype: 'ToDo',
        name: _todo.value.name,
        fieldname: todoData,
      })
      toast.success(__('ToDo updated'))
    } else {
      // Create new
      await call('frappe.client.insert', {
        doc: {
          doctype: 'ToDo',
          ...todoData,
        },
      })
      toast.success(__('ToDo created'))
    }

    emit('save', _todo.value)
  } catch (e) {
    error.value = __('Failed to save ToDo')
    console.error(e)
  }
}

async function handleGoogleCalendarSync() {
  const eventData = {
    subject: 'ToDo: ' + (_todo.value.title || __('No Title')),
    description: _todo.value.description || '',
    starts_on: _todo.value.date + ' 09:00',
    ends_on: _todo.value.date + ' 10:00',
    all_day: true,
    status: 'Open',
    event_type: 'Private',
    sync_with_google_calendar: true,
    google_calendar: _linkedEvent.value.googleCalendar,
    event_participants: eventParticipants.value.map((email) => ({
      reference_doctype: 'User',
      reference_docname: 'Guest',
      email: email,
    })),
  }

  if (_linkedEvent.value.name) {
    // Update existing event
    await call('frappe.client.set_value', {
      doctype: 'Event',
      name: _linkedEvent.value.name,
      fieldname: eventData,
    })
  } else {
    // Create new event
    const result = await call('frappe.client.insert', {
      doc: {
        doctype: 'Event',
        ...eventData,
      },
    })
    if (result?.name) {
      _linkedEvent.value.name = result.name
    }
  }
}

function deleteToDo() {
  $dialog({
    title: __('Delete ToDo'),
    message: __('Are you sure you want to delete this ToDo?'),
    actions: [
      {
        label: __('Delete'),
        variant: 'solid',
        theme: 'red',
        onClick: async (closeDialog) => {
          try {
            if (_todo.value.name) {
              await call('frappe.client.delete', {
                doctype: 'ToDo',
                name: _todo.value.name,
              })
              toast.success(__('ToDo deleted'))
            }
            emit('delete', _todo.value.id)
            closeDialog()
          } catch (e) {
            toast.error(__('Failed to delete ToDo'))
            console.error(e)
          }
        },
      },
    ],
  })
}

function close() {
  show.value = false
  emit('close')
}

// Keyboard shortcuts
useKeyboardShortcuts({
  active: show,
  shortcuts: [
    {
      keys: ['Escape'],
      action: close,
    },
    {
      keys: ['Enter'],
      guard: () => props.mode === 'details',
      action: switchToEdit,
    },
    {
      keys: ['Delete', 'Backspace'],
      guard: () => props.mode === 'details' || props.mode === 'edit',
      action: deleteToDo,
    },
  ],
})

defineExpose({
  updateToDo: (updates) => {
    Object.assign(_todo.value, updates)
  },
})
</script>
