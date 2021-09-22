<template>
<div id="app">
    <div class="esquerda">
        <h3>Protótipo de Editor em Tempo Real (React Native/ Expo)</h3>
        <h4>
            Modifique o código abaixo e visualize as alterações em tempo real no seu
            celular
        </h4>
        <MonacoEditor v-if="isShow" @change="atualizarCodigo" theme="vs-dark" :diffEditor="false" language="javascript" :original="codigo" class="editor" :value="codigo" v-model="codigo"></MonacoEditor>
        <textarea style="width:595px;height:50px" v-model="dependencias" placeholder='{"expo-status-bar":{"version":"~1.0.4"}}'></textarea>
        <br />
        <button @click="atualizarDependencias()">Atualizar dependências</button>
        {{ msg }}
        <h4>VueJS / Monaco Editor / Expo Snack SDK</h4>
    </div>

    <div class="direita">
        <h3>Aponte a câmera do seu celular para o QR CODE</h3>
        <qrcode-vue :value="url" :size="size" level="H" v-if="url" />

        <div style="">Dispositivos conectados: {{ dispositivos }}</div>

        <div>Versão EXPO: {{ SDK }}</div>
        <h4>Desenvolvido por Daniel Melo Alencar</h4>
        <a href="https://expo.dev/client" target="blank">Baixe o app cliente aqui
        </a>
    </div>
</div>
</template>

<script>
import {
    Snack
} from "snack-sdk";

import QrcodeVue from "qrcode.vue";
import MonacoEditor from "vue-monaco"; //https://github.com/egoist/vue-monaco
export default {
    name: "App",
    components: {
        QrcodeVue,
        MonacoEditor,
    },
    data() {
        return {
            isShow: false,
            SDK: 0,
            dispositivos: 0,
            snack: "",
            url: null,
            size: 200,
            id: "",
            msg: "",
            dependencias: `{
"expo-status-bar":{"version":"~1.0.4"}
}
`,
            codigo: `
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';
const styles = StyleSheet.create({
  container: {
    flex:1,
    paddingTop: 50,
    justifyContent:'center',
    alignItems:'center' 
  },
});

export default function App() {
   return (
    <View style={styles.container}>
      <Text style={{fontSize:30}} >This is magic </Text>
    </View>
  );
}      
 `,
        };
    },
    methods: {
        async pegaURL() {
            // Create Snack
            this.snack = new Snack({
                online: true,
                codeChangesDelay: 500,
                name: "Meu App",
                dependencies: {},
            });

            // Make the Snack available online
            this.snack.setOnline(true);

            this.snack.getStateAsync().then((response) => {
                this.url = response.url;
                this.SDK = response.sdkVersion;
            });

            this.snack.addStateListener((state, prevState) => {
                if (state.connectedClients !== prevState.connectedClients) {
                    this.dispositivos = Object.keys(state.connectedClients).length;
                    for (const key in state.connectedClients) {
                        if (!prevState.connectedClients[key]) {
                            /* console.log(
                                 "A client has connected! " +
                                 state.connectedClients[key].platform
                             );
                             */
                        }
                    }
                }
            });
        },

        async atualizarCodigo() {
            this.snack.updateFiles({
                "App.js": {
                    type: "CODE",
                    contents: this.codigo,
                },
            });
            /*
            this.snack.getStateAsync().then((response) => {
                console.log(response);
            });
            */

            /*
             const {
                 id
             } = await this.snack.saveAsync();            
             */
            //console.log(`Saved with id ${id}`);

            await this.snack.saveAsync();

            /*
            const url = await this.snack.getDownloadURLAsync();
            console.log("Download URL: " + url);
            */
        },

        async pegaInfo() {

            //await this.snack.setName('MEUAPP')
            this.snack.getStateAsync().then(() => {
                // this.url = response.url;
                //console.log(response);
            });
        },
        async atualizarDependencias() {

            this.msg = "Atualizando dependências...";
            this.snack.updateDependencies(JSON.parse(this.dependencias.trim()));
            const {
                dependencies
            } = await this.snack.getStateAsync();
            var erro = false;
            for (const name in dependencies) {
                if (dependencies[name].handle === undefined) {
                    erro = true;
                    alert("Não foi possível resolver a dependencia " + name);
                    //console.log(dependencies[name].handle);
                }
            }
            /** RESOLVE AS DEPENDENCIAS PERDIDAS */

            const depend = {};
            for (const name in this.snack.getState().missingDependencies) {
                depend[name] = {
                    version: this.snack.getState().missingDependencies[name]
                        .wantedVersion,
                };
                this.msg = "Instalando ependencia perdida: " + name;
            }
            //console.log(depend);
            this.snack.updateDependencies(depend);

            /**************************** */

            this.msg = erro ? "Erro" : "Concluído";
        },
    },
    mounted() {
        this.$nextTick(() => {
            this.isShow = true;
        });
        this.pegaURL();
    },
};
</script>

<style scoped>
#app {
    display: flex;
    font-family: "Tahoma", Courier, monospace;
    color: rgb(92, 92, 92);
}

.esquerda {
    flex: 1;
}

.direita {
    display: flex;
    flex-direction: column;
    flex: 1;
    align-items: center;
    justify-content: center;
}

.editor {
    width: 600px;
    height: 350px;
}
</style>
