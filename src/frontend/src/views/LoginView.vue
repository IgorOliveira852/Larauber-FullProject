<template>
    <div class="pt-16">
        <h1 class="text-3x1 font-bold mb-4">Digite seu número de telefone</h1>
        <form v-if="!waitingOnVerification" action="#" @submit.prevent="handleLogin">
            <div class="overflow-hidden shadow sm:rounded-md max-w-sm mx-auto text-left">
                <div class="bg-white px-4 py-5 sm:p-6">
                    <div>
                        <input 
                            type="text" v-maska data-maska="+## (##) # ####-####" v-model="credentials.phone" name="phone" id="phone" placeholder="+55 (12) 3 4567-8910"
                            class="mt-1 block w-full px-3 py-2 rounded-md border border-gray-300 shadow-sm focus:border-black focus:outline-none">
                    </div>
                </div>
                <div class="bg-gray-50 px-4 py-3 text-right sm:px-6">
                    <button type="submit"  @submit.prevent="handleLogin"
                        class="inline-flex justify-center rounded-md border border-transparent bg-black py-2 px-4 text-sm font-medium text-white shadow-sm hover:bg-gray-600 focus:outline-none">Continue</button>
                </div>
            </div>
        </form>
        <form v-else action="#" @submit.prevent="handleVerification">
            <div class="overflow-hidden shadow sm:rounded-md max-w-sm mx-auto text-left">
                <div class="bg-white px-4 py-5 sm:p-6">
                    <div>
                        <input 
                            type="text" v-maska data-maska="######" v-model="credentials.login_code" name="login_code" id="login_code" placeholder="123456"
                            class="mt-1 block w-full px-3 py-2 rounded-md border border-gray-300 shadow-sm focus:border-black focus:outline-none">
                    </div>
                </div>
                <div class="bg-gray-50 px-4 py-3 text-right sm:px-6">
                    <button type="submit"  @submit.prevent="handleVerification"
                        class="inline-flex justify-center rounded-md border border-transparent bg-black py-2 px-4 text-sm font-medium text-white shadow-sm hover:bg-gray-600 focus:outline-none">Continue</button>
                </div>
            </div>
        </form>
    </div>
</template>

<script setup>
import { vMaska } from 'maska'
import { ref, reactive, onMounted } from 'vue'
import axios from 'axios'
import { useRouter } from 'vue-router';

const router = useRouter()

const credentials = reactive({
    phone: null
})

const waitingOnVerification = ref(false)

onMounted(() => {
    if (localStorage.getItem('token')) {
        router.push({
            name: 'landing'
        })
    }
})

const getFormattedCredentials = () => {
    return {
        phone: credentials.phone.replaceAll(' ', '').replace('(', '').replace(')', '').replace('-', ''),
        login_code: credentials.login_code
    }
}

const handleLogin = () => {
    axios.post('http://127.0.0.1:8000/api/login', getFormattedCredentials())
        .then((response) => {
            console.log(response.data)
            waitingOnVerification.value = true
        })
        .catch((error) => {
            console.error(error)
            alert(error.response.data.message)
        })
    }

    const handleVerification = () => {
        axios.post('http://127.0.0.1:8000/api/login/verify', getFormattedCredentials())
            .then((response) => {
                console.log(response.data); // auth token
                localStorage.setItem('token', response.data)
                router.push({
                    name: 'landing'
                })
            })
            .catch((error) => {
                console.error(error);
                alert(error.response.data.message)
            })
    }
</script>