<template>
    <div class="wrapper" id="wrapper">
        <div class="check-sheet" id="check-sheet">
            <div class="check-front" id="check-front">
                <div class="check-front-print" id="check-box-print">
                <div class="account-holder-name" style="position: absolute; top: 40px; left: 60px">{{check.accountHolderName}}</div>
                <div class="account-holder-address" style="position: absolute; top: 70px; left: 60px">
                    {{check.accountHolderAddress}}<br>
                    {{check.accountHolderCity}}, {{check.accountHolderState}} {{check.accountHolderZip}}
                </div>
                <div class="check-number-human" style="position: absolute; top: 40px; left: 1060px">{{check.checkNumber}}</div>
                <div class="date-data" style="position: absolute; top: 80px; left: 850px">{{check.date}}</div>
                <div class="date" style="position: absolute; top: 90px; left: 760px">Date: _____________________ </div>
                <div class="amount-box" style="position: absolute; top: 175px; left: 950px">

                </div>
                <div class="amount-data" style="position: absolute; top: 182px; left: 970px">{{formatMoney(check.amount)}}</div>
                <div class="pay-to-data" style="position: absolute; top: 180px; left: 180px">{{check.payTo}}</div>
                <div class="pay-to" style="position: absolute; top: 170px; left: 60px">
                    Pay to the <br>Order of <span class="payto-line"></span>
                </div>
                <div class="amount-line-data" ref="line" style="position: absolute; top: 240px; left: 100px">
                    ***
                    {{toWords(check.amount)}} 
                    ***
                </div>
                <div class="amount-line" style="position: absolute; top: 250px; left: 60px">
                    <span class="dollar-line"></span>
                </div>
                <div class="bank-name" style="position: absolute; top: 300px; left: 60px">{{check.bankName}}</div>
                <div class="memo-data" style="position: absolute; top: 367px; left: 120px">{{check.memo}}</div>
                <div class="memo" style="position: absolute; top: 390px; left: 60px">
                    Memo: ____________________________________
                </div>
                <div class="signature-data" style="position: absolute; top: 366px; left: 770px">{{check.signature}}</div>
                <div class="signature" style="position: absolute; top: 390px; left: 750px">
                    _________________________________________________
                </div>
                <div class="banking" style="position: absolute; top: 420px; left: 80px">
                    <div class="routing" style="display: inline;">
                        a{{check.routingNumber}}a
                    </div>
                    <div class="bank-account" style="display: inline;">{{check.bankAccountNumber}}c</div>
                    <div class="check-number" style="display: inline; margin-left:20px">{{check.checkNumber}}</div>
                </div>
                </div>
            </div>
            <div class="check-cut-gap"></div>
            <div class="check-back" id="check-back">
                <img src="@/assets/check-back.svg" class="check-back-template" alt="">
                <div class="back-endorsement-data">
                    <div v-for="line in backEndorsementLines" :key="line">{{ line }}</div>
                </div>
            </div>
        </div>
        <div class="check-data">
            <div class="alert alert-primary" role="alert"><strong>Background does not print.</strong></div>
            <div style="float: right;">
                <button type="button" class="btn btn-primary" style="margin-right: 10px;" @click="printCheck('front')">Print Front</button>
                <button type="button" class="btn btn-primary" @click="printCheck('back')">Print Back</button>
            </div>
            <form class="row g-3">
                <div class="col-md-6">
                    <label for="inputEmail4" class="form-label">Account Holder Name</label>
                    <input type="email" class="form-control" id="inputEmail4" v-model="check.accountHolderName">
                </div>
                <div class="col-md-6">
                </div>
                <div class="col-md-4">
                    <label for="inputAddress" class="form-label">Address</label>
                    <input type="text" class="form-control" id="inputAddress" v-model="check.accountHolderAddress">
                </div>
                <div class="col-md-2">
                    <label for="inputCity" class="form-label">City</label>
                    <input type="text" class="form-control" v-model="check.accountHolderCity">
                </div>
                <div class="col-md-2">
                    <label for="inputState" class="form-label">State</label>
                    <input type="text" class="form-control" v-model="check.accountHolderState">
                </div>
                <div class="col-md-2">
                    <label for="inputZip" class="form-label">Zip</label>
                    <input type="text" class="form-control" v-model="check.accountHolderZip">
                </div>
            </form>
            <form class="row g-3" style="margin-top: 30px; border-top: 1px solid #e7e7e7;">
                <div class="col-md-2">
                    <label for="inputEmail4" class="form-label">Check Number</label>
                    <input type="email" class="form-control" id="inputEmail4" v-model="check.checkNumber">
                </div>
                <div class="col-md-4">
                    <label for="inputAddress" class="form-label">Bank Name</label>
                    <input type="text" class="form-control" id="inputAddress" v-model="check.bankName">
                </div>
                <div class="col-md-2">
                    <label for="inputCity" class="form-label">Routing #</label>
                    <input type="text" class="form-control" v-model="check.routingNumber">
                </div>
                <div class="col-md-2">
                    <label for="inputState" class="form-label">Account #</label>
                    <input type="text" class="form-control" v-model="check.bankAccountNumber">
                </div>
                <div class="col-md-6">
                    <label for="inputZip" class="form-label">Memo</label>
                    <input type="text" class="form-control" v-model="check.memo">
                </div>
            </form>
            <form class="row g-3" style="margin-top: 30px; border-top: 1px solid #e7e7e7;">
                <div class="col-md-2">
                    <label for="inputEmail4" class="form-label">Amount</label>
                    <input type="email" class="form-control" id="inputEmail4" v-model="check.amount">
                </div>
                <div class="col-md-6">
                    <label for="inputZip" class="form-label">Pay To</label>
                    <input type="text" class="form-control" v-model="check.payTo">
                </div>
                <div class="col-md-2">
                    <label for="inputEmail4" class="form-label">Date</label>
                    <input type="email" class="form-control" id="inputEmail4" v-model="check.date">
                </div>
                <div class="col-md-6">
                    <label for="inputZip" class="form-label">Signature</label>
                    <input type="text" class="form-control" v-model="check.signature">
                </div>
                <div class="col-md-2">
                    <div class="form-check" style="margin-top: 32px;">
                        <input
                            class="form-check-input"
                            type="checkbox"
                            id="mobileDeposit"
                            :checked="check.endorsementMode === 'mobile'"
                            :disabled="check.endorsementMode === 'custom'"
                            @change="updateEndorsementMode('mobile', $event)"
                        >
                        <label class="form-check-label" for="mobileDeposit">Mobile</label>
                    </div>
                </div>
                <div class="col-md-2">
                    <div class="form-check" style="margin-top: 32px;">
                        <input
                            class="form-check-input"
                            type="checkbox"
                            id="customEndorsement"
                            :checked="check.endorsementMode === 'custom'"
                            :disabled="check.endorsementMode === 'mobile'"
                            @change="updateEndorsementMode('custom', $event)"
                        >
                        <label class="form-check-label" for="customEndorsement">Endorse</label>
                    </div>
                </div>
                <div class="col-md-6" v-if="check.endorsementMode === 'custom'">
                    <label for="endorsementText" class="form-label">Endorsement</label>
                    <input
                        type="text"
                        class="form-control"
                        id="endorsementText"
                        v-model="check.endorsementText"
                    >
                </div>
            </form>
            <div class="col-12" style="margin-top: 30px;">
                <button type="button" class="btn btn-primary" @click="saveToHistory">Save to History</button>
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
import { ToWords } from 'to-words';
import { computed, ref, reactive, nextTick, watch, onMounted, onUnmounted } from 'vue'
import { formatMoney } from '../utilities'
import { useAppStore } from '../stores/app'

type EndorsementMode = 'none' | 'mobile' | 'custom'

type CheckData = {
    accountHolderName: string
    accountHolderAddress: string
    accountHolderCity: string
    accountHolderState: string
    accountHolderZip: string
    checkNumber: string
    date: string
    bankName: string
    amount: string
    payTo: string
    memo: string
    signature: string
    routingNumber: string
    bankAccountNumber: string
    endorsementMode: EndorsementMode
    endorsementText: string
    lineLength?: number
}

type SavedCheckData = Partial<CheckData>

function getEndorsementMode(value: unknown): EndorsementMode {
    return value === 'mobile' || value === 'custom' ? value : 'none'
}

const state = useAppStore()

const toWordsTool = new ToWords({
  localeCode: 'en-US',
  converterOptions: {
    currency: true,
    ignoreDecimal: false,
    ignoreZeroCurrency: false,
    doNotAddOnly: true,
  },
});

const toWords: (denom: number | string) => string = (denom) => {
    try {
        return toWordsTool.convert(Number(denom), );
    } catch (e) {
        return `${e}`;
    }
}

function printCheck (_side?: 'front' | 'back') {
    const style = document.createElement('style');
    style.textContent = `
      @media print {
        @page {
          margin: 0;
        }
        body {
          transform: scale(1);
          transform-origin: top center;
          width: 149%;
          margin: 0;
          padding: 0;
        }
        .wrapper > *:not(.check-box) {
          display: none !important;
        }
        .check-data {
            display: none;
        }
        .check-box {
          position: fixed;
          top: 0;
          left: 0;
          width: 100%;
          height: 100%;
          margin: 0;
          padding: 0px;
          background-color: white;
          background: white !important;
          border: none !important;
          box-shadow: none !important;
        }
        .check-box-print {
          position: relative;
        }
      }
    `;
    document.head.appendChild(style);
    window.print();
    style.remove();
}

function saveToHistory () {
    let checkList = JSON.parse(localStorage.getItem('checkList') || '[]')
    checkList.push({ ...check })
    localStorage.setItem('checkList', JSON.stringify(checkList))
}

function genNewCheck (): CheckData {
    const checkList = JSON.parse(localStorage.getItem('checkList') || '[]') as SavedCheckData[]
    const recentCheck = checkList[checkList.length - 1]
    return {
        accountHolderName: recentCheck?.accountHolderName || 'John Smith',
        accountHolderAddress: recentCheck?.accountHolderAddress || '123 Cherry Tree Lane',
        accountHolderCity: recentCheck?.accountHolderCity || 'New York',
        accountHolderState: recentCheck?.accountHolderState || 'NY',
        accountHolderZip: recentCheck?.accountHolderZip || '10001',
        checkNumber: recentCheck?.checkNumber ? `${parseInt(recentCheck.checkNumber, 10) + 1}` : '100',
        date: new Date().toLocaleDateString(),
        bankName: recentCheck?.bankName || 'Bank Name, INC',
        amount: '0.00',
        payTo: 'Michael Johnson',
        memo: recentCheck?.memo || 'Rent',
        signature: recentCheck?.signature || 'John Smith',
        routingNumber: recentCheck?.routingNumber || '022303659',
        bankAccountNumber: recentCheck?.bankAccountNumber || '000000000000',
        endorsementMode: getEndorsementMode(recentCheck?.endorsementMode),
        endorsementText: recentCheck?.endorsementText || '',
    }
}

const check = reactive(
    genNewCheck()
)

const line = ref<HTMLElement | null>(null)

const backEndorsementLines = computed(() => {
    if (check.endorsementMode === 'mobile') {
        return ['For Mobile Deposit Only', check.signature].filter(Boolean)
    }

    if (check.endorsementMode === 'custom') {
        return [check.endorsementText, check.signature].filter(Boolean)
    }

    return [check.signature].filter(Boolean)
})

watch(check, async () => {
    await nextTick(() => {
        let computedLine = line?.value?.clientWidth
        check.lineLength = computedLine
    })
}, { immediate: true })

function updateEndorsementMode(mode: EndorsementMode, event: Event) {
    const checked = (event.target as HTMLInputElement).checked
    check.endorsementMode = checked ? mode : 'none'
}

function handlePrintShortcut(event: KeyboardEvent) {
    if (event.ctrlKey && event.key === 'p') {
        event.preventDefault();
        printCheck();
    }
}

onMounted(() => {
    const savedCheck = state.check as SavedCheckData | null
    if (savedCheck) {
        check.accountHolderName = savedCheck.accountHolderName ?? check.accountHolderName
        check.accountHolderAddress = savedCheck.accountHolderAddress ?? check.accountHolderAddress
        check.accountHolderCity = savedCheck.accountHolderCity ?? check.accountHolderCity
        check.accountHolderState = savedCheck.accountHolderState ?? check.accountHolderState
        check.accountHolderZip = savedCheck.accountHolderZip ?? check.accountHolderZip
        check.checkNumber = savedCheck.checkNumber ?? check.checkNumber
        check.date = savedCheck.date ?? check.date
        check.bankName = savedCheck.bankName ?? check.bankName
        check.amount = savedCheck.amount ?? check.amount
        check.payTo = savedCheck.payTo ?? check.payTo
        check.memo = savedCheck.memo ?? check.memo
        check.signature = savedCheck.signature ?? check.signature
        check.routingNumber = savedCheck.routingNumber ?? check.routingNumber
        check.bankAccountNumber = savedCheck.bankAccountNumber ?? check.bankAccountNumber
        check.endorsementMode = getEndorsementMode(savedCheck.endorsementMode)
        check.endorsementText = savedCheck.endorsementText ?? ''
    }
    state.check = null

    window.addEventListener('keydown', handlePrintShortcut);
});

onUnmounted(() => {
    window.removeEventListener('keydown', handlePrintShortcut);
});

</script>

<style>

label {
    font-weight: bold;
}
.memo-data {
    font-family: Caveat;
    font-size: 30px;
    max-width: 350px;
    line-height: 0.65;
}
.signature-data {
    font-family: Caveat;
    font-size: 40px;
    transform: rotate(-2deg);
}
.amount-line-data {
    text-transform: capitalize;
}
.date-data, .pay-to-data, .amount-data{
    font-size: 20px;
    font-weight: bold;
}
.check-data {
    margin-top: 50px;
    padding: 50px 120px;
    border-top: 1px solid #e6e6e6;
}
.bank-name{
    font-size: 20px;
    font-weight: bold;
}
.account-holder-name {
    font-size: 20px;
    font-weight: bold;
}
.check-number-human {
    font-size: 20px;
    font-weight: bold;
}
.amount-box::before {
    content: "$";
    font-size: 20px;
    margin-left: -15px;
    font-weight: bold;
}
.amount-box {
    width: 225px;
    height: 40px;
    border: 1px solid #c7c7c7;
    background-color: white;
}
.check-box {
    width: 1200px;
    height: 1553px;
    border: 1px solid #e6e6e6;
    background-color: white;
    margin: 0 auto;
    background: url('../assets/checkbg.png');
    background-repeat: no-repeat;
    background-size: contain;
}

#check-box {
    width: 100%;
}

@font-face {
    font-family: 'banking';
    src: url('../assets/micrenc.ttf');
}


.banking {
    font-family: 'banking';
    font-size: 37px;
}
.dollar-line::after{
    content: "Dollars";
    font-size: 18px;
    position: absolute;
    right: -73px;
    top: 0;
}
.dollar-line {
    width: 840px;
    display: block;
    border-bottom: 1px solid black;
    margin-left: 10px;
    margin-top: 20px;
}
.payto-line {
    width: 776px;
    display: block;
    border-bottom: 1px solid black;
    margin-left: 73px;
    border-right: 1px solid black;
    height: 28px;
    margin-top: -32px;
}
</style>
