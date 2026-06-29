<template>
	<ion-page>
		<ion-content class="ion-padding">
			<div
				v-if="resetPassword.showDialog"
				class="flex h-screen w-screen flex-col bg-white"
			>
				<header class="flex items-center justify-between px-6 py-4">
					<div class="text-lg font-semibold text-gray-900">
						{{ __("Reset Password") }}
					</div>
					<button
						type="button"
						class="text-sm text-gray-600 hover:text-gray-900 underline"
						@click="resetPassword.showDialog = false"
					>
						{{ __("Back to Login") }}
					</button>
				</header>
				<div class="flex flex-1 flex-col items-center justify-center px-8 text-center">
					<p class="text-gray-700">
						{{ __("Your password has expired. Please reset your password to continue") }}
					</p>
					<a
						class="mt-6 inline-flex items-center justify-center gap-2 transition-colors focus:outline-none text-white bg-gray-900 hover:bg-gray-800 active:bg-gray-700 focus-visible:ring focus-visible:ring-gray-400 h-9 text-base px-4 rounded"
						:href="resetPassword.link"
						target="_blank"
					>
						{{ __("Go to Reset Password page") }}
					</a>
				</div>
			</div>

			<div v-else class="flex h-screen w-screen flex-col justify-center bg-white">
				<div class="flex flex-col mx-auto gap-3 items-center">
					<FrappeHRLogo class="h-8 w-8" />
					<div class="text-3xl font-semibold text-gray-900 text-center">
						{{ __("Login to Frappe HR") }}
					</div>
				</div>

				<div class="mx-auto mt-10 w-full px-8 sm:w-96">
					<form v-if="!user_pass_login_disabled.data" class="flex flex-col space-y-4" @submit.prevent="submit">
						<Input
							:label="__('Email')"
							:placeholder="__('johndoe@mail.com')"
							v-model="email"
							type="text"
							autocomplete="username"
						/>
						
						<!-- Password dengan toggle -->
						<div class="relative">
							<Input
								:label="__('Password')"
								:type="showPassword ? 'text' : 'password'"
								placeholder="••••••"
								v-model="password"
								autocomplete="current-password"
							/>
							<button
								type="button"
								class="absolute right-3 top-7 text-gray-500 hover:text-gray-700 focus:outline-none"
								@click="showPassword = !showPassword"
							>
								<FeatherIcon :name="showPassword ? 'eye-off' : 'eye'" class="h-5 w-5" />
							</button>
						</div>
						
						<div class="flex justify-end mt-1">
							<a
								href="/#forgot"
								target="_blank"
								class="text-sm text-gray-600 hover:text-gray-900 transition-colors"
							>
								{{ __("Forgot Password?") }}
							</a>
						</div>
						<ErrorMessage :message="errorMessage" />
						<Button
							:loading="session.login.loading"
							variant="solid"
							class="disabled:bg-gray-700 disabled:text-white !mt-6"
						>
							{{ __("Login") }}
						</Button>
						<div class="text-center mt-4">
							<router-link
								:to="{ name: 'ForgotPassword', query: email ? { email } : {} }"
								class="text-sm text-gray-600 hover:text-gray-900 underline"
							>
								{{ __("Forgot Password?") }}
							</router-link>
						</div>
					</form>

					<template v-if="authProviders.data?.length">
						<div v-if="!user_pass_login_disabled.data" class="text-center text-sm text-gray-600 my-4">or</div>
						<div class="space-y-4">
							<a
								v-for="provider in authProviders.data"
								:key="provider.name"
								class="flex items-center justify-center gap-2 transition-colors focus:outline-none text-gray-800 bg-gray-100 hover:bg-gray-200 active:bg-gray-300 focus-visible:ring focus-visible:ring-gray-400 h-7 text-base p-2 rounded"
								:href="provider.auth_url"
							>
								<img class="h-4 w-4" :src="provider.icon" :alt="provider.provider_name" />
								<span>Login with {{ provider.provider_name }}</span>
							</a>
						</div>
					</template>

					<div v-else-if="user_pass_login_disabled.data" class="text-center text-gray-600 py-8">{{ __("No login methods are available. Please contact your administrator.") }}</div>
				</div>
			</div>

			<!-- Dialog tetap sama -->
			<Dialog v-model="resetPassword.showDialog">
				<!-- ... -->
			</Dialog>

			<Dialog v-model="otp.showDialog">
				<!-- ... -->
			</Dialog>
		</ion-content>
	</ion-page>
</template>

<script setup>
import { IonPage, IonContent } from "@ionic/vue"
import { inject, reactive, ref } from "vue"
import { Input, Button, ErrorMessage, Dialog, createResource, FeatherIcon } from "frappe-ui"

import FrappeHRLogo from "@/components/icons/FrappeHRLogo.vue"

const email = ref(null)
const password = ref(null)
const showPassword = ref(false)
const errorMessage = ref("")

const resetPassword = reactive({
	showDialog: false,
	link: "",
})
const otp = reactive({
	showDialog: false,
	tmp_id: "",
	code: "",
	verification: {},
})

const session = inject("$session")
const __ = inject("$translate")

async function submit(e) {
	try {
		let response
		if (otp.showDialog) {
			response = await session.otp(otp.tmp_id, otp.code)
		} else {
			response = await session.login(email.value, password.value)
		}

		if (response.message === "Password Reset") {
			resetPassword.showDialog = true
			resetPassword.link = response.redirect_to
		} else {
			resetPassword.showDialog = false
			resetPassword.link = ""
		}

		if (response.verification) {
			if (response.verification.setup) {
				otp.showDialog = true
				otp.tmp_id = response.tmp_id
				otp.verification = response.verification
			} else {
				window.open("/login?redirect-to=" + encodeURIComponent(window.location.pathname), "_blank")
			}
		}
	} catch (error) {
		errorMessage.value = error.messages.join("\n")
	}
}

const user_pass_login_disabled = createResource({
	url: "hrms.api.system_settings.get_user_pass_login_disabled",
	method: 'GET',
	initialData: 1,
	auto: true,
})

const authProviders = createResource({
	url: "hrms.api.oauth.oauth_providers",
	auto: true,
})
</script>