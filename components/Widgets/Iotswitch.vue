<template>
    <card>
        <template slot="header">           
            <h5 class="card-category"> {{ config.selectedDevice.name }} - {{ config.variableFullName }}</h5>
            <h3 class="card-title">
                <i
                  class="fa " :class="[config.icon, getIconColorClass()]" style="font-size: 30px"
                ></i>
                <base-switch @click="value = !value; sendValue()" :value="value" type="primary" on-text="ON" off-text="OFF" style="margin-top: 10px;" class="pull-right">
                </base-switch>
            </h3>
        </template>
    </card>
</template>


<script>
    export default {
        name: 'iotswitch',
        props: ['config'],
        
        data() {
            return {
                value: true,
                valueact: false,
                topic: "",
                props: ['config']   
            };
        },
        watch: {
            config: {
                immediate: true,
                deep: true,
                handler() {
                    setTimeout(() => {
                        this.valueact = false;

                        this.$nuxt.$off(this.topic);

                        //userId/dId/uniquestr/sdata
                        const topic = this.config.userId + "/" + this.config.selectedDevice.dId + "/" + this.config.variable + "/sdata";
                        this.$nuxt.$on(topic, this.processReceivedData);

                    }, 300);

                }
            }
        },

        mounted() {
            const topic = this.config.userId + "/" + this.config.selectedDevice.dId + "/" + this.config.variable + "/sdata";
            this.$nuxt.$on(topic, this.processReceivedData);
        },

        beforeDestroy() {
            this.$nuxt.$off(this.topic);
        },
        methods: {
            processReceivedData(data){
                try {
                     console.log("received");
                     console.log(data);
                     this.valueact = data.valueact;
                  } catch (error) {
                     console.log(error);
                     }
                 },
            getIconColorClass() {
                //para apagar el icono 
                if (!this.valueact){
                    return "text-dark";
                }

                if (this.config.class == "success") {
                    return "text-success";
                }
                if (this.config.class == "primary") {
                    return "text-primary";
                }
                if (this.config.class == "warning") {
                    return "text-warning";
                }
                if (this.config.class == "danger") {
                    return "text-danger";
                }
            },


            sendValue(){

                const toSend = {
                    topic: this.config.userId + '/' + this.config.selectedDevice.dId + '/' + this.config.variable + '/actdata',
                    msg: {
                        value: this.value
                    }
                };

                $nuxt.$emit('mqtt-sender', toSend);

            }


        }
    };
</script>
<style></style>