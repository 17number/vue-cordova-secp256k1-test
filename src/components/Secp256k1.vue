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
  generateKeyPair() {
    do {
      this.privKey = randomBytes(32);
    } while(!secp256k1.privateKeyVerify(this.privKey));
    this.pubKey = secp256k1.publicKeyCreate(this.privKey);
  }
  async initSecp256k1() {
    this.secp256k1Entity = await instantiateSecp256k1();
  }
  hashMessage() {
    this.msgHashed = Buffer.from(cryptoJs.SHA256(this.msg).toString(), 'hex');
    this.signAndVerifyBtcTs();
    this.signAndVerifySecp256k1Node();
  }
  signAndVerifyBtcTs() {
    this.signBtcTs = this.secp256k1Entity.signMessageHashCompact(this.privKey, this.msgHashed);
    this.signBtcTsStr = Buffer.from(this.signBtcTs).toString('hex');
    this.verifidBtcTs = this.secp256k1Entity.verifySignatureCompact(this.signBtcTs, this.pubKey, this.msgHashed);
  }
  signAndVerifySecp256k1Node() {
    const sigObj = secp256k1.sign(this.msgHashed, this.privKey);
    this.signSecp256k1Node = sigObj.signature.toString('hex');
    this.verifidSecp256k1Node = secp256k1.verify(this.msgHashed, sigObj.signature, this.pubKey);
  }
  created() {
    this.generateKeyPair();
    this.initSecp256k1();
  }
}
</script>

<style scoped lang="scss">
</style>
