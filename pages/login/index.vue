<template>
    <div class="flex">
        <div class="w-1/2 h-screen relative">
            <div class="h-full w-full bg-black opacity-20 absolute top-0 left-0"></div>
            <img src="~/assets/image/login/login_side_background_image.png" alt="bg-image" class="object-cover h-full w-full">
        </div>
        <div class="w-1/2 h-screen bg-white flex justify-center items-center">
            <div class="w-[440px] rounded-md p-4">
                <div class="w-36 h-36 mx-auto">
                    <img src="~/assets/image/login/logo.png" alt="logo" class="w-full h-full object-cover">
                </div>
                <h1 class="text-center mt-12 font-bold text-primary text-3xl">ເຂົ້າສູ່ລະບົບ</h1>
                <p class="text-center text-gray-500 mt-3">ປ້ອນເບີ ແລະ ລະຫັດຜ່ານຂອງທ່ານເພື່ອເຂົ້າສູ່ລະບົບ</p>
                <n-form
                ref="formRef"
                :label-width="80"
                :model="formValue"
                :rules="rules"
                :size="size"
                >
                    <n-form-item label="ອີເມລ" path="email" class="mt-16">
                        <n-input 
                        placeholder="ປ້ອນອີເມລ..." 
                        v-model:value="formValue.email" 
                        @keydown.enter.prevent
                        :disabled="loading"
                        />
                    </n-form-item>
                    <n-form-item path="password" label="ລະຫັດຜ່ານ" class="mt-2">
                        <n-input
                        v-model:value="formValue.password"
                        type="password"
                        show-password-on="click"
                        placeholder="ປ້ອນລະຫັດຜ່ານ..."
                        :maxlength="20"
                        class="w-full"
                        :disabled="loading"
                        >
                            <template #password-visible-icon>
                                <n-icon :size="16" :component="EyeOutline" />
                            </template>
                            <template #password-invisible-icon>
                                <n-icon :size="16" :component="EyeOffOutline" />
                            </template>
                        </n-input>
                    </n-form-item>
                </n-form>
                <n-button type="primary" class="w-full mt-12 mb-8" color="#002749" @click="handleLogin" :loading="loading">
                    ເຂົ້າສູ່ລະບົບ
                </n-button>
            </div>
        </div>
    </div>
</template>

<script setup>

definePageMeta({
  layout: 'blank'
})

import { EyeOffOutline, EyeOutline } from '@vicons/ionicons5'
import { useMessage } from 'naive-ui';

const message = useMessage();

const formRef = ref(null);
const size = ref('medium');
const formValue = ref({
    email: "southixa.pele10@gmail.com",
    password: "",
})

const rules = {
  email: {
    required: true,
    validator(rule, value) {
        if(!value) {
            return new Error("ກະລຸນາໃສ່ອີເມລ")
        } else if (!(/@gmail.com/i.test(value))){
            return new Error("ກະລຸນາໃສ່ @gmail.com")
        }
        return true
    },
    trigger: 'blur',
  },
  password: {
    required: true,
    message: 'ກະລຸນາໃສ່ລະຫັດຜ່ານ',
    trigger: 'blur',
  }
};

const loading = ref(false);

const cookie = useCookie('token')

import { NhostClient } from "@nhost/vue";
import { onMounted } from 'vue';

const runtimeConfig = useRuntimeConfig();

const nhost = new NhostClient({
    subdomain: runtimeConfig.public.NHOST_SUBDOMAIN,
    region: runtimeConfig.public.NHOST_REGION
});

async function handleLogin(e) {
    e.preventDefault();
    try{
        loading.value = true;
        const warining = await formRef.value?.validate().catch((error)=>{return error;})
        if(warining.length > 0) {
            message.error("ຂໍ້ມູນບໍ່ຖືກຕ້ອງ")
            console.log(warining)
            throw new Error('invalid input => ' + warining);
        }

        //await nhost.auth.signOut();
        const res = await nhost.auth.signIn(formValue.value)
        if(res.error){
            message.error("ຂໍ້ມູນບໍ່ຖືກຕ້ອງ")
            throw new Error('invalid input => ' + error)
        }
        message.success("ເຂົ້າສູ່ລະບົບສຳເລັດ")
        cookie.value = res.session.accessToken;
        loading.value = false;
        await navigateTo({ path: '/dashboard' });

    } catch (error) {
        loading.value = false;
        console.log("error occured in handleLogin method => ", error);
    }
}


onMounted(async () => {
    cookie.value = "";
    await nhost.auth.signOut();
})

</script>