<template>
    <section>
		<section class="box contain w640 add-new-chain desktop-only">
			<h2 class="center-text" style="width:100%;">{{ $t('lang.submitNewChain') }}</h2>
			<section class="input-container">
				<input placeholder="http://chaindomain.com/" v-model="newChain" />
				<button @click="addChain">{{ $t('lang.addChain') }}</button>
			</section>

		</section>
		<hr class="desktop-only" />

		<section v-if="chains.length">
			<section class="contain" style="margin-top:20px;">
				<h2 v-html="$t('lang.selectChain')"></h2>
				<p v-html="$t('lang.selectChainInfo')"></p>
			</section>
			<hr/>

			<section class="contain">
				<table>
					<thead>
					<tr>
						<th>{{ $t('lang.chainId') }}</th>
						<th class="desktop-only">{{ $t('lang.producers') }}</th>
						<th class="desktop-only">{{ $t('lang.addedDate') }}</th>
						<th></th>
					</tr>
					</thead>


					<tbody>
					<tr :key="chain.chainId" v-for="chain in chains">
						<td><b>id:{{chain.id}}</b> - {{chain.chainId.substr(0,15)}}...</td>
						<td class="desktop-only">{{getProducerCount(chain)}}</td>
						<td class="desktop-only">{{new Date(chain.createdAt*1000).toLocaleDateString()}}</td>
						<td><router-link :to="'chain/'+chain.id.toString()" tag="a">
							<button>{{ $t('lang.select') }}</button>
						</router-link></td>
					</tr>
					</tbody>
				</table>
			</section>
		</section>


		<section class="contain" v-else>
			<h1>{{ $t('lang.noChains') }}</h1>
			<h2>{{ $t('lang.noChainsInfo') }}</h2>
		</section>
    </section>
</template>

<script lang="ts">
	import {Component, Vue, Watch} from "vue-property-decorator";
import { mapState, mapActions } from "vuex";
import Eos from 'eosjs';
import * as urlUtils from "@/utils/url.util";
import * as api from "@/api";
	import {getChainProducers, getChainState, getProducerCount} from "@/utils/eos.util";
import {addChain} from '@/api'

@Component({
	components: {},
	computed: mapState(["chains"]),
	methods: mapActions(["getChains"]),
	data(){return {
		newChain:''
	}}
})
export default class Chains extends Vue {
	chains!: any[];
	getChains!: () => void;
	producerCounts:Array<{chainId:string, count:number}> = [];

	newChain:string = '';

	created() {
		this.getChains();
		this.newChain = '';
	}

	async addChain(){
		if(this.newChain.indexOf('http://') != 0 && this.newChain.indexOf('https://') != 0)
			return alert('Malformed EOSIO node URL');

		const {host, port} = urlUtils.urlToHostPort(this.newChain);

		const eos = Eos({httpEndpoint:`http://${host}:${port}`});
		const info = await eos.getInfo({}).catch(() => null);

		if(!info || typeof info !== 'object' || !info.hasOwnProperty('head_block_num'))
			return alert('Could not get chain info');

		// TODO: Add feedback
		await addChain(this.newChain);
	}

	getProducerCount(chain:any){
		const count:any = this.producerCounts.find((x:any) => x.chainId === chain.chainId);
		return count ? count.count : 0;
	}

	@Watch('chains')
	async chainsChanged(){
		this.chains.map(async chain => {
			const producers:number = await getProducerCount(chain.url);
			this.producerCounts.push({chainId:chain.chainId, count:producers});
		})
	}
}
</script>

<style lang="scss">
	.add-new-chain {
		margin:50px auto;
		/*box-shadow: inset 0 0 150px rgba(0,0,0,0.3);*/

		.input-container {
			margin-top:15px;
		}
	}
</style>
