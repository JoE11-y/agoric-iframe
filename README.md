# Agoric Iframe Sandbox

A standalone, isolated iframe package for Agoric blockchain integration. This package runs all Agoric SDK code in a sandboxed environment, preventing dependency conflicts with your main application.

## Features

- **Zero Dependency Conflicts**: All Agoric dependencies are isolated
- **Standalone Bundle**: Single HTML file with all JS bundled
- **SES Lockdown Applied**: Hardens JavaScript built-ins before any Agoric code runs
- **Safe Isolation**: SES runs in iframe, preventing conflicts with main app
- **Simple Integration**: Drop-in iframe solution
- **Multiple Deployment Options**: npm package, CDN, or direct file usage

## Quick Start

### Option 1: Install as NPM Package

```bash
npm install @qstn/agoric-iframe-sandbox
# or
yarn add @qstn/agoric-iframe-sandbox
```

Then copy the built iframe to your public directory:

```bash
cp node_modules/@qstn/agoric-iframe-sandbox/dist/agoric-sandbox.html public/
```

### Option 2: Build from Source

```bash
git clone <repository>
cd agoric-iframe-sandbox
npm install
npm run build
```

The built files will be in `dist/`:
- `agoric-sandbox.html` - The iframe HTML file
- `agoric-sandbox.[hash].js` - The bundled JavaScript (referenced by HTML)

### Option 3: Use from CDN (jsDelivr + GitHub)

```bash
# After building and pushing to GitHub, use jsDelivr CDN
# URL format: https://cdn.jsdelivr.net/gh/user/repo@version/file

# Example: Load from a specific release/tag
https://cdn.jsdelivr.net/gh/qstn/agoric-iframe-sandbox@v1.0.0/dist/agoric-sandbox.html

# Or from a branch
https://cdn.jsdelivr.net/gh/qstn/agoric-iframe-sandbox@main/dist/agoric-sandbox.html
```

Configure in your main app:
```bash
# .env
REACT_APP_AGORIC_IFRAME_URL=https://cdn.jsdelivr.net/gh/qstn/agoric-iframe-sandbox@v1.0.0/dist/agoric-sandbox.html
```

## Usage in Your App

### 1. Add the iframe to your public directory

Copy `dist/agoric-sandbox.html` to your app's public folder.

### 2. Use the iframe handler

The parent app communicates with the iframe via postMessage:

```typescript
import { useAgoric } from './hooks/use-agoric';

function PaymentButton() {
  const { fundSurvey, isConnected } = useAgoric();

  const handlePay = async () => {
    try {
      const result = await fundSurvey({
        surveyId: 'survey-123',
        amount: '1000000',
        denom: 'ubld'
      });
      console.log('Payment successful:', result.txHash);
    } catch (err) {
      console.error('Payment failed:', err);
    }
  };

  return <button onClick={handlePay}>Pay with Agoric</button>;
}
```

## API

The iframe responds to these message types:

### `CONNECT_WALLET`
Connect to Keplr wallet.

**Request:**
```javascript
{ type: 'CONNECT_WALLET', id: 'unique-id' }
```

**Response:**
```javascript
{
  type: 'AGORIC_RESPONSE',
  id: 'unique-id',
  success: true,
  data: { address: 'agoric1...' }
}
```

### `FUND_SURVEY`
Fund a survey with tokens.

**Request:**
```javascript
{
  type: 'FUND_SURVEY',
  id: 'unique-id',
  data: {
    surveyId: 'survey-123',
    amount: '1000000',
    denom: 'ubld'
  }
}
```

**Response:**
```javascript
{
  type: 'AGORIC_RESPONSE',
  id: 'unique-id',
  success: true,
  data: {
    success: true,
    txHash: '0x...',
    height: 12345
  }
}
```

### `CLAIM_REWARDS`
Claim rewards from a survey.

**Request:**
```javascript
{
  type: 'CLAIM_REWARDS',
  id: 'unique-id',
  data: {
    surveyId: 'survey-123',
    userId: 'user-456'
  }
}
```

### `GET_STATUS`
Get current connection status.

**Request:**
```javascript
{ type: 'GET_STATUS', id: 'unique-id' }
```

**Response:**
```javascript
{
  type: 'AGORIC_RESPONSE',
  id: 'unique-id',
  success: true,
  data: {
    initialized: true,
    connected: true,
    address: 'agoric1...',
    hasBrands: true,
    brandsAvailable: ['BLD', 'IST']
  }
}
```

## Development

### Build for production
```bash
npm run build
```

### Watch mode (rebuilds on changes)
```bash
npm run watch
```

### Dev server (opens browser with iframe)
```bash
npm run dev
```

## Configuration

Edit `src/index.js` to change network configuration:

```javascript
const CONFIG = {
  CHAIN_ID: "agoricdev-25",
  RPC_ENDPOINT: "https://devnet.rpc.agoric.net:443",
  REST_ENDPOINT: "https://devnet.api.agoric.net",
  NETWORK_CONFIG_HREF: "https://devnet.agoric.net/network-config",
};
```

## Customization

### Add Custom Contract Interactions

Edit the `fundSurvey` and `claimRewards` functions in `src/index.js`:

```javascript
async function fundSurvey({ surveyId, amount, denom }) {
  // Your custom contract logic here
  const invitationSpec = {
    source: "contract",
    instance: YOUR_CONTRACT_INSTANCE,
    publicInvitationMaker: "yourInvitationMaker",
  };

  // ... rest of implementation
}
```

## Bundle Size

- **Total bundle**: ~3-4 MB (uncompressed), ~800 KB (gzipped)
- **Main dependencies**: @agoric/web-components, @agoric/rpc, ses

The large bundle size is due to the Agoric SDK. The bundle is only loaded when the iframe is created (lazy loading recommended).

## Browser Support

- Chrome/Edge: ✅ Latest 2 versions
- Firefox: ✅ Latest 2 versions
- Safari: ✅ Version 12+
- Mobile: ⚠️ Limited (Keplr extension required)

## Security

### Origin Validation

In production, add origin validation to `src/index.js`:

```javascript
window.addEventListener("message", async event => {
  const allowedOrigins = ['https://yourdomain.com'];
  if (!allowedOrigins.includes(event.origin)) {
    console.error('[Agoric Sandbox] Invalid origin:', event.origin);
    return;
  }
  // ... rest of handler
});
```

### Content Security Policy

Ensure your CSP allows same-origin iframes:

```
Content-Security-Policy: frame-src 'self';
```

## Troubleshooting

### "Keplr not found"
- Install Keplr browser extension: https://www.keplr.app/

### "Sandbox failed to load"
- Check browser console for errors
- Verify the iframe HTML file is accessible
- Check network tab for failed requests

### Bundle size too large
- The Agoric SDK is large (~3-4 MB)
- Consider lazy loading the iframe only when needed
- Use CDN with caching for production

## License

MIT

## Support

For issues and questions:
- GitHub Issues: <repository-url>
- Documentation: See parent app docs
