ai-agent/src/index.ts

import express from 'express';
import { SolanaAgent } from '@solana-agent-kit/core';
import { mintPhilosophyNFT } from './nft-mint';
import { getLunqBalance } from './solana-client';

const app = express();
app.use(express.json());

// Initialize AI Agent
const agent = new SolanaAgent({
  rpcUrl: process.env.SOLANA_RPC!,
  wallet: JSON.parse(process.env.WALLET_KEYPAIR!),
});

// Philosophical response endpoint
app.post('/ask', async (req, res) => {
  try {
    const { message, userWallet } = req.body;
    
    // Check LQ balance
    const balance = await getLunqBalance(userWallet);
    if (balance < 1000) throw new Error('Minimum 1000 LQ required');

    // Generate AI response
    const aiResponse = await agent.executeAction('philosophy', message);
    
    // Mint NFT
    const nftTx = await mintPhilosophyNFT(aiResponse, userWallet);

    res.json({
      answer: aiResponse,
      nftTx: nftTx
    });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

app.listen(3000, () => {
  console.log('AI Agent running on port 3000');
});
