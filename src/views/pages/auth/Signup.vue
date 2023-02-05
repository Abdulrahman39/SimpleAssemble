<script setup>
import { useLayout } from '@/layout/composables/layout';
import { ref, computed } from 'vue';
import AppConfig from '@/layout/AppConfig.vue';
import axios from 'axios'
import { useRouter } from 'vue-router'
const { layoutConfig, contextPath } = useLayout();
const email = ref('');
const password = ref('');
const checked = ref(false);
const router = useRouter()
const invalidClass = ref('');
const isWrong = ref({
    email: {
        class: 'w-full mb-3 ',
        msg: false,

    }
});


const isEmpty = ref({
    firstname: {
        class: 'w-full md:w-30rem ',
        msg: false
    },

    lastname: {
        class: 'w-full md:w-30rem ',
        msg: false
    },
    email: {
        class: 'w-full md:w-30rem ',
        msg: false,
        MSG: ''
    },
    password: {
        class: 'w-full md:w-30rem ',
        msg: false
    }
});

function checkData() {
    let flag = 0;

    if (password.value.length < 8) {
        isEmpty.value.password.msg = true;
        isEmpty.value.password.class = 'w-full md:w-30rem p-invalid';
        flag = 1;
    }
    if (password.value.length >= 8) {
        isEmpty.value.password.msg = false;
        isEmpty.value.password.class = 'w-full md:w-30rem ';
    }
    if (email.value == '') {
        isEmpty.value.email.msg = true;
        isEmpty.value.email.MSG = 'This field is required';
        isEmpty.value.email.class = 'w-full md:w-30rem p-invalid';
        flag = 1;
    } else {
        isEmpty.value.email.msg = false;
        isEmpty.value.email.class = 'w-full md:w-30rem ';
    }
    if (flag == 0) return true;
    else return false;
}


async function addUser(){
    const headers = {
                'Authorization': window.localStorage.getItem("Token"),
                'Content-Type': 'application/json',
            };
            const data = {
              uid: window.localStorage.getItem("uid")

            }
    await axios.post(`https://simple-assemble-af150-default-rtdb.firebaseio.com/users/users.json`,data).then(() => {});
}
async function signup(e) {
    e.preventDefault();
    if (checkData()) {
        const data = {

            email: email.value,
            password: password.value,
            returnSecureToken: true,
        }
        const config = {
            headers: {
                'Content-Type': 'application/x-www-form-urlencoded',
            },
        };

        const res = await axios.post('https://identitytoolkit.googleapis.com/v1/accounts:signUp?key=AIzaSyAYb6qJoUAV9MDpYtO1ITQZ57v0TIq8vWI', data, config).catch((error) => {
            invalidClass.value = 'p-invalid';
            isWrong.value.email.class = 'w-full md: w-30rem mb-5 p-invalid';
            isWrong.value.email.msg = true;
            console.log(error)
        });

        if (res.status == 200) {
            window.localStorage.setItem('Token', res.data.idToken);
            window.localStorage.setItem('uid', res.data.localId);
            await addUser();
            router.push('/');
        }


    }
}

const logoUrl = computed(() => {
    return `${contextPath}layout/images/${layoutConfig.darkTheme.value ? 'logo-white' : 'logo-dark'}.svg`;
});
</script>

<template>
    <div
        class="surface-ground flex align-items-center justify-content-center min-h-screen min-w-screen overflow-hidden">
        <div class="flex flex-column align-items-center justify-content-center">
            <div
                style="border-radius: 56px; padding: 0.3rem; background: linear-gradient(180deg, var(--primary-color) 10%, rgba(33, 150, 243, 0) 30%)">
                <div class="w-full surface-card py-6 px-3 sm:px-8" style="border-radius: 53px">
                    <div class="text-center mb-3">
                        <h2>Sign up</h2>
                    </div>

                    <div>

                        <label for="email1" class="block text-900 text-l font-medium mb-1">Email</label>

                        <InputText id="email1" type="text" placeholder="Email address" :class="isEmpty.email.class"
                            v-model="email" />
                        <div class="fieldBox mb-5 mt-1">
                            <InlineMessage v-if="isEmpty.email.msg">{{ isEmpty.email.MSG }}</InlineMessage>
                        </div>
                        <label for="password1" class="block text-900 font-medium text-l mb-1">Password</label>
                        <Password id="password1" v-model="password" placeholder="Password" :toggleMask="true"
                            :class="isEmpty.password.class" inputClass="w-full"><template #header>
                                <h6>Pick a password</h6>
                            </template>
                            <template #footer="sp">
                                {{ sp.level }}
                                <Divider />
                                <p class="mt-2">Suggestions</p>
                                <ul class="pl-2 ml-2 mt-0" style="line-height: 1.5">
                                    <li>At least one lowercase</li>
                                    <li>At least one uppercase</li>
                                    <li>At least one numeric</li>
                                    <li>Minimum 8 characters</li>
                                </ul>
                            </template>
                        </Password>
                        <div v-if="isEmpty.password.msg" class="fieldBox mt-1">
                            <InlineMessage>Weak Password</InlineMessage>
                        </div>
                        <div class="flex align-items-center justify-content-between mb-5 gap-5"></div>
                        <p>
                            Already have an account?
                            <router-link class="text-primary" :to="{ name: 'login' }">log in</router-link>
                        </p>
                        <Button @click="signup" type="submit" label="Sign Up" class="w-full p-3 text-l"></Button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<style scoped>
.pi-eye {
    transform: scale(1.6);
    margin-right: 1rem;
}

.pi-eye-slash {
    transform: scale(1.6);
    margin-right: 1rem;
}
</style>
