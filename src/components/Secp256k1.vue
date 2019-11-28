<template>
  <div class="secp256k1">
    <div>
      <label for="msg">message</label>
      <input name="msg" type="text" v-model="msg" @input="hashMessage">
    </div>
    <div>
      <h4>hashed</h4>
      <p>{{ msgHashed.toString('hex') }}</p>
    </div>
    <div>
      <h4>signature(btc-ts)</h4>
      <p>{{ signBtcTsStr }}</p>
      <h4>valid(btc-ts)</h4>
      <p>{{ verifidBtcTs }}</p>
    </div>
    <div>
      <h4>signature(secp256k1-node)</h4>
      <p>{{ signSecp256k1Node }}</p>
      <h4>valid(secp256k1-node)</h4>
      <p>{{ verifidSecp256k1Node }}</p>
    </div>
    <div>
      <h3>Times</h3>
      <h4>generateKeyPair</h4>
      <p>{{ times.generateKeyPair }}</p>
      <h4>initSecp256k1</h4>
      <p>{{ times.initSecp256k1 }}</p>
      <h4>signAndVerifyBtcTs</h4>
      <p>{{ times.signAndVerifyBtcTs }}</p>
      <h4>signAndVerifySecp256k1Node</h4>
      <p>{{ times.signAndVerifySecp256k1Node }}</p>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";
import { instantiateSecp256k1 } from "bitcoin-ts";
const { randomBytes } = require("crypto");
const cryptoJs = require("crypto-js");
const secp256k1 = require("secp256k1");
const Buffer = require('buffer/').Buffer;

@Component
export default class Secp256k1 extends Vue {
  privKey = null;
  pubKey = null;
  msg = 'hello';
  msgHashed = '';
  secp256k1Entity: any = null;
  signBtcTs = '';
  signBtcTsStr = '';
  verifidBtcTs = false;
  signSecp256k1Node = '';
  verifidSecp256k1Node = false;
  times = {
    generateKeyPair: -1,
    initSecp256k1: -1,
    signAndVerifyBtcTs: -1,
    signAndVerifySecp256k1Node: -1,
  };
  generateKeyPair() {
    const s = new Date();
    do {
      this.privKey = randomBytes(32);
    } while(!secp256k1.privateKeyVerify(this.privKey));
    this.pubKey = secp256k1.publicKeyCreate(this.privKey);
    const e = new Date();
    this.times.generateKeyPair = e.getTime() - s.getTime();
  }
  async initSecp256k1() {
    const s = new Date();
    this.secp256k1Entity = await instantiateSecp256k1().catch();
    const e = new Date();
    this.times.initSecp256k1 = e.getTime() - s.getTime();
  }
  hashMessage() {
    this.msgHashed = Buffer.from(cryptoJs.SHA256(this.msg).toString(), 'hex');
    this.signAndVerifySecp256k1Node();
    this.signAndVerifyBtcTs();
  }
  signAndVerifyBtcTs() {
    const s = new Date();
    this.signBtcTs = this.secp256k1Entity.signMessageHashCompact(this.privKey, this.msgHashed);
    this.signBtcTsStr = Buffer.from(this.signBtcTs).toString('hex');
    this.verifidBtcTs = this.secp256k1Entity.verifySignatureCompact(this.signBtcTs, this.pubKey, this.msgHashed);
    const e = new Date();
    this.times.signAndVerifyBtcTs = e.getTime() - s.getTime();
  }
  signAndVerifySecp256k1Node() {
    const s = new Date();
    const sigObj = secp256k1.sign(this.msgHashed, this.privKey);
    this.signSecp256k1Node = sigObj.signature.toString('hex');
    this.verifidSecp256k1Node = secp256k1.verify(this.msgHashed, sigObj.signature, this.pubKey);
    const e = new Date();
    this.times.signAndVerifySecp256k1Node = e.getTime() - s.getTime();
  }
  created() {
    this.generateKeyPair();
    this.initSecp256k1();
  }
}
</script>

<style scoped lang="scss">
</style>
