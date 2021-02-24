<template>
    <h1 class="text-xl lg:text-5xl pb-6">{{ msg }}</h1>
    <div id="watcher">
        <input class="w-full p-8 bg-gray-200 border-current rounded-md mb-12" placeholder="Søk på navn eller org.nummer" v-model="firm">

        <table v-if="typeof data === 'object'" class="table-auto max-w-full overflow-auto border-collapse border border-gray-300  block rounded-md">
            <thead>
                <tr>
                    <th class="border border-gray-300 px-4">Organisasjonsnavn</th>
                    <th class="border border-gray-300 px-4">Organisasjonsnummer</th>
                    <th class="border border-gray-300 px-4">Adresse</th>
                    <th class="border border-gray-300 px-4">Postnummer</th>
                    <th class="border border-gray-300 px-4">Poststed</th>
                    <th class="border border-gray-300 px-4">Hjemmeside</th>
                </tr>
            </thead>
            <tbody>
                <tr v-if="(typeof data.page != 'undefined')" v-for="item in data._embedded.enheter" v-bind:class = "(item.konkurs)?'konkurs bg-red-200':''">
                    <td class="border border-gray-300 px-4">{{item.navn}}</td>
                    <td class="border border-gray-300 px-4">{{item.organisasjonsnummer}}</td>

                    <td class="border border-gray-300 px-4">
                        <p v-if="(typeof item.postadresse != 'undefined')" v-for="adressepunkt in item.postadresse.adresse">{{adressepunkt}}</p>
                        <p v-else-if="(typeof item.forretningsadresse != 'undefined')" v-for="adressepunkt in item.forretningsadresse.adresse">{{adressepunkt}}</p>
                        <p v-else> - </p>
                    </td>
                    <td class="border border-gray-300 px-4">
                        <p v-if="(typeof item.postadresse != 'undefined')">{{item.postadresse.postnummer}}</p>
                        <p v-else-if="(typeof item.forretningsadresse != 'undefined')">{{item.forretningsadresse.postnummer}}</p>
                        <p v-else> - </p>
                    </td>
                    <td class="border border-gray-300 px-4">
                        <p v-if="(typeof item.postadresse != 'undefined')">{{item.postadresse.poststed}}</p>
                        <p v-else-if="(typeof item.forretningsadresse != 'undefined')">{{item.forretningsadresse.poststed}}</p>
                        <p v-else> - </p>
                    </td>


                    <td v-if="item.hjemmeside != null" class="border border-gray-300 px-4">
                        <a :href="'https://' + item.hjemmeside" class="hover:text-blue-600">{{item.hjemmeside}}</a>
                    </td>
                </tr>

                <tr v-if="(typeof data.page == 'undefined')" v-bind:class = "(data.konkurs)?'konkurs bg-red-200':''">
                    <td class="border border-gray-300 px-4">{{data.navn}}</td>
                    <td class="border border-gray-300 px-4">{{data.organisasjonsnummer}}</td>

                    <td class="border border-gray-300 px-4">
                        <p v-if="(typeof data.postadresse != 'undefined')" v-for="adressepunkt in data.postadresse.adresse">{{adressepunkt}}</p>
                        <p v-else-if="(typeof data.forretningsadresse != 'undefined')" v-for="adressepunkt in data.forretningsadresse.adresse">{{adressepunkt}}</p>
                        <p v-else> - </p>
                    </td>
                    <td class="border border-gray-300 px-4">
                        <p v-if="(typeof data.postadresse != 'undefined')">{{data.postadresse.postnummer}}</p>
                        <p v-else-if="(typeof data.forretningsadresse != 'undefined')">{{data.forretningsadresse.postnummer}}</p>
                        <p v-else> - </p>
                    </td>
                    <td class="border border-gray-300 px-4">
                        <p v-if="(typeof data.postadresse != 'undefined')">{{data.postadresse.poststed}}</p>
                        <p v-else-if="(typeof data.forretningsadresse != 'undefined')">{{data.forretningsadresse.poststed}}</p>
                        <p v-else> - </p>
                    </td>

                    <td v-if="data.hjemmeside != null" class="border border-gray-300 px-4">
                        <a :href="'https://' + data.hjemmeside" class="hover:text-blue-600">{{data.hjemmeside}}</a>
                    </td>
                </tr>
            </tbody>
        </table>
        <p v-if="typeof data === 'string'"> {{data}} </p> 
    </div>
</template>

<script>
    import _ from 'lodash'
    import axios from 'axios'
    
    export default {
        props: ["msg"],
        el: '#watcher',
        data(){
            return {
                firm: '',
                data: '',
            }
        },
        watch: {
            firm: function (newFirm, oldFirm) {
                this.data = 'Avventer innhold..'
                this.debouncedGetData()
            }
        },
        created(){
            this.debouncedGetData = _.debounce(this.getData, 400)
        },
        methods: {
            getData() {
                if (this.firm.length == 0) {
                    this.data = ''
                    return
                }

                var query = ''
                var orgnum = false

                if (isNaN(this.firm)) {
                    if (this.firm.length < 3) {
                        this.data = 'Fyll i tre tegn for å søke på organisasjonsnavn'
                        return
                    }
                    query = 'https://data.brreg.no/enhetsregisteret/api/enheter?navn='
                } else {
                    if (this.firm.length < 9) {
                        this.data = 'Fyll i et fullstendig organisasjonsnummer'
                        return
                    }
                    query = 'https://data.brreg.no/enhetsregisteret/api/enheter/'
                    orgnum = true
                }
                
                this.data = 'Søker..'
                var vm = this
                
                axios.get(query + this.firm)
                    .then(function (response) {
                        console.log(response)
                        if (!orgnum && response.data.page.totalElements == 0) {
                            vm.data = 'Ingen treff. Prøv igjen'
                            return
                        }
                        vm.data = response.data
                    })
                    
                    .catch(function (error) {
                        if (orgnum) {
                            vm.data = 'Oppfør gyldig organisasjonsnummer! ' + error
                            return
                        }
                        vm.data = 'Error! Kan ikke nå APIet. ' + error
                    })
            }
        }
    }
</script>