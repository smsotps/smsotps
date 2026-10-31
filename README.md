# SMS OTPS - Receive SMS Online with Temporary Numbers

[SMS OTPS](https://smsotps.com) is an online service to receive SMS online with temporary non-VoIP phone numbers. Use a virtual number to receive OTP and SMS verification codes when signing up for apps and websites - no physical SIM card needed, on site or via API.

Whether you need a disposable number for a single sign-up or an API to automate verification in your own software, SMS OTPS gives you a simple way to get a phone number online and read the incoming SMS on the site.

## What you can do

- **One-time number leases** - a temporary number for a single SMS verification. Pick a service and a country, get a number, and receive the code on the site.
- **Developer API** - automate SMS verification in your own apps with a REST API: check offers, order numbers, read the SMS, cancel or resend from code.
- **Bulk link orders** - order link-based verification numbers in quantity through the same API.
- **Free test numbers** - try receiving SMS online with public free numbers before you create an order.

## How it works

1. Create an account at smsotps.com
2. Choose a service and country
3. Get your temporary phone number
4. Receive the SMS code on the site or through the API

You see the price before you buy, and the cost of an order is taken from your wallet when you order. Wallet top-ups are made with crypto.

Numbers are real SIM-based, non-VoIP phone numbers, so they work with services that block VoIP ranges.

## Popular uses

- Verifying accounts on messaging and social apps
- Testing apps that require phone verification
- Protecting your personal number from spam and marketing lists
- Creating separate accounts for work, testing or privacy without sharing your own SIM
- Bulk SMS verification through the API

## Services, countries and languages

The catalog covers popular messaging apps, social networks, email providers and marketplaces, with numbers in many countries. Prices differ by service and country and are shown live on the pricing page.

The website is available in several languages, including English, Russian, Chinese, Spanish, French, German, Portuguese, Arabic, Hindi, Turkish, Ukrainian, Vietnamese, Indonesian, Italian and Polish.

## For developers

REST API with number ordering, status checks, cancel and resend. Authenticate with an API key sent in the `X-API-KEY` header, and call `https://api.smsotps.com/api`.

A minimal example that orders a number:

```bash
curl -X POST https://api.smsotps.com/api/order-number \
  -H "X-API-KEY: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"service":"fb","country":1}'
```

Keep your API key on your server and never publish it in browser code or a public repository. See [API.md](API.md) in this repo or the full documentation at https://smsotps.com/apidocs

## Frequently asked questions

**How do I receive an SMS online with SMS OTPS?**
Create an account, choose the service and country, and order a number. Enter the number on the service you are verifying. The SMS code appears on the site and through the API.

**What is a temporary phone number?**
It is a phone number you use for a short time to receive a verification SMS, so you do not have to give out your personal number.

**Do I need a physical SIM card?**
No. The numbers are virtual and work entirely online. You only need an account.

**Will the numbers work with services that block VoIP?**
The numbers are real SIM-based, non-VoIP numbers, so they work with services that block VoIP ranges.

**Can I use SMS OTPS from my own software?**
Yes. Use the REST API described in API.md.

**Are there free numbers?**
Yes, the free numbers page lists public numbers you can use to try receiving SMS online.

**How do I contact support?**
Email support@smsotps.com or use the contact page on the website.

## Links

- Website: https://smsotps.com
- Free numbers: https://smsotps.com/free-numbers
- Pricing: https://smsotps.com/pricing
- API documentation: https://smsotps.com/apidocs

Support: support@smsotps.com
