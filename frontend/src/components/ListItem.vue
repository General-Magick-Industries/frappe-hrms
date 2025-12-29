<template>
	<div class="flex flex-col w-full justify-center gap-2.5">
		<div class="flex flex-row items-center justify-between">
			<div class="flex flex-row items-start gap-3 grow">
				<slot name="left" />
			</div>
			<div class="flex flex-row justify-end items-center gap-2">
				<slot name="right" />
			</div>
		</div>
		<div v-if="props.isTeamRequest" class="flex flex-row items-center gap-2 pl-8">
			<EmployeeAvatar :employeeID="props.employee" />
			<div class="text-sm text-gray-600 grow">
				<!-- {{ props.employeeName }} -->
				{{ displayName }}
			</div>
		</div>
	</div>
</template>

<script setup>
// Option 1
// import EmployeeAvatar from "@/components/EmployeeAvatar.vue"

// Option 2
import { computed } from "vue"
import { watch } from "vue"
import EmployeeAvatar from "@/components/EmployeeAvatar.vue"
import { getEmployeeInfo } from "@/data/employees"

const props = defineProps({
	isTeamRequest: {
		type: Boolean,
		default: false,
	},
	employee: {
		type: String,
		required: false,
	},
	employeeName: {
		type: String,
		required: false,
	},
	customNickname: {
		type: String,
		required: false,
	},
})

// const employeeData = computed(() => getEmployeeInfo(props.employee))

// const displayName = computed(() => {
// 	return employeeData.value?.custom_nickname || props.employeeName
// })



const employeeData = computed(() => getEmployeeInfo(props.employee))

// Debug: lihat isi employeeData
watch(employeeData, (val) => {
	console.log("Employee ID:", props.employee)
	console.log("Employee Data:", val)
	console.log("Custom Nickname:", val?.custom_nickname)
}, { immediate: true })

const displayName = computed(() => {
	return employeeData.value?.custom_nickname || props.employeeName
})
</script>
