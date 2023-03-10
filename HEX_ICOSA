"use strict";
/* jshint node: true, esversion: 8 */
const axios = require('axios');
const { BigNumber } = require('bignumber.js');
const Web3EthAbi = require('web3-eth-abi');

async function rpcEthCall(rpcURL, params) {
    const payload = {
        "id": 1,
        "jsonrpc": "2.0",
        "method": "eth_call",
        "params": [
            params,
            "latest"
        ]
    };
    //console.log("SENDING", params);

    const result = await axios.post(rpcURL, payload);
    return result.data;
}

async function main() {
    const config = {
        hexStakeAddress: '0x1e18ed1bfCa02e59a43de32c60Ac0FD4923b64b5', // WIZ Treasury
        icosaContractAddress: '0xfc4913214444af5c715cc9f7b52655e788a569ed',
    };

    const rpcURL = `https://eth.llamarpc.com`;

    const encData = await rpcEthCall(
        rpcURL,
        {
            "data": '0x86a99e4a' + Web3EthAbi.encodeParameters(['address'], [config.hexStakeAddress]).substr(2),
            "to": config.icosaContractAddress,
        }
    );

    const stakeData = Web3EthAbi.decodeParameters(
      [
         { 'type': 'uint64', name: 'stakeStart' },
         { 'type': 'uint64', name: 'capitalAdded' },
         { 'type': 'uint120', name: 'stakePoints' },
         { 'type': 'bool', name: 'isActive' },
         { 'type': 'uint80', name: 'payoutPreCapitalAddIcsa' },
         { 'type': 'uint80', name: 'payoutPreCapitalAddHdrn' },
         { 'type': 'uint80', name: 'stakeAmount' },
         { 'type': 'uint16', name: 'minStakeLength' },
      ],
      encData.result
    );
    //console.log(stakeData);

    const stakeEndEncData = await rpcEthCall(
        rpcURL,
        {
            "data": "0x900d58de" /* icsaStakeEnd() */,
            "from": config.hexStakeAddress,
            "to": config.icosaContractAddress,
        }
    );
    const stakeEndData = Web3EthAbi.decodeParameters(
      [
         { 'type': 'uint256', name: 'payoutIcsaPlusBonusIcsa' },
         { 'type': 'uint256', name: 'payoutHdrn' },
         { 'type': 'uint256', name: 'principalPenalty' },
         { 'type': 'uint256', name: 'payoutPenaltyIcsa' },
         { 'type': 'uint256', name: 'payoutPenaltyHdrn' },
      ],
      stakeEndEncData.result
    );
    //console.log(stakeEndData);
  
    const stakeAmount = BigNumber(stakeData.stakeAmount).toNumber() / 1e9;
    console.log("ICSA Staked", stakeAmount);

    const payoutIcsaPlusBonusIcsa = BigNumber(stakeEndData.payoutIcsaPlusBonusIcsa).toNumber() / 1e9;
    const payoutHdrn = BigNumber(stakeEndData.payoutHdrn).toNumber() / 1e9;
    const payoutPenaltyIcsa = BigNumber(stakeEndData.payoutPenaltyIcsa).toNumber() / 1e9;
    const payoutPenaltyHdrn = BigNumber(stakeEndData.payoutPenaltyHdrn).toNumber() / 1e9;
    const principalPenalty = BigNumber(stakeEndData.principalPenalty).toNumber() / 1e9;
    console.log('ICSA Yield', payoutIcsaPlusBonusIcsa + payoutPenaltyIcsa);
    console.log('HDRN Yield', payoutHdrn + payoutPenaltyHdrn);
}

main().then(() => process.exit(0)).catch(err => { console.log(err); process.exit(6); });
