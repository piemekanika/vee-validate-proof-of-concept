<template>
    <div>
        <validation-observer v-slot="{ validate }" ref="form">
            <div>
                Нужно ввести число между {{ model.foo }} и {{ model.bar }}      
                
                <br><br><br>      
            </div>        
        
            <div v-if="isLoading">
                Loading...
            </div>
            <div>
                <validation-provider v-slot="{ errors }" :rules="{ asyncCheck: model }">
                    <input type="text" :disabled="isLoading" v-model="model.userInput">
                    
                    <button @click="submit" :disabled="isLoading">submit</button>
                    
                    <div v-if="!isLoading && errors.length">
                        {{ errors }}
                    </div>
                </validation-provider>
            </div>
        </validation-observer>
    </div>
</template>

<script>
import { extend, ValidationProvider, ValidationObserver } from 'vee-validate';
import Vue from 'vue';

Vue.component('ValidationProvider', ValidationProvider);
Vue.component('ValidationObserver', ValidationObserver);

const fakeApiCall = model => new Promise((res, rej) => {
    setTimeout(() => {
        const userInput = Number(model.userInput);
        
        if (userInput < model.bar && userInput > model.foo) {
            res();
        } else {
            rej(new Error('Uga buga'));
        }
    }, 1000);
})

extend('asyncCheck', {
    validate: async (val, { model }) => {
        console.log(val, model);
        try {
            await fakeApiCall(model);
           
            return true;
        } catch (e) {
            return 'Seems you did something wrong :(';
        }
    },
    params: ['model']
});

export default {
    data() {
        return {
            model: {
                userInput: '',
                foo: 15,
                bar: 100,
            },
            isLoading: false,
        };
    },
    methods: {
        async submit() {
            
            this.isLoading = true;
            
            const success = await this.$refs.form.validate();
            
            if (success) {
                alert('success');
            }
            
            this.isLoading = false;
        },
    },
}
</script>
