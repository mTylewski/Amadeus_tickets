# ✈️ Flight Price Monitor - n8n Workflow

Automated flight price monitoring system with Telegram notifications. This workflow searches for flights from Polish cities to popular European island destinations and sends alerts about attractive deals.

## 🚀 Features

- ✈️ Automatic flight search from selected Polish airports
- 🏝️ Monitor popular island destinations (Crete, Malta, Majorca, etc.)
- 💰 Filter by maximum flight price
- 📱 Telegram notifications for attractive deals
- 🔥 Deal categorization: Super Deal / Good Deal / Too Expensive
- 🔄 Flexible date and destination configuration
- 💎 Smart price analysis with savings calculation
- ⏱️ Rate limiting to respect API quotas

## 🛠️ Requirements

### API Keys & Accounts
1. **Amadeus API** - Register at [developers.amadeus.com](https://developers.amadeus.com)
2. **Telegram Bot** - Create bot via [@BotFather](https://t.me/BotFather)
3. **n8n** - Running n8n instance

### Configuration Data Needed
- `amadeusClientId` - Client ID from Amadeus API
- `amadeusClientSecret` - Client Secret from Amadeus API
- `telegramChatId` - Your Telegram Chat ID
- `rapidApiKey` - RapidAPI key (for future hotel/motorcycle features)

## 📋 Setup Instructions

### 1. Get Telegram Chat ID
Send a message to your bot, then visit:
```
https://api.telegram.org/bot<YOUR_BOT_TOKEN>/getUpdates
```
Find your `chat.id` in the response.

### 2. Configure Amadeus API
1. Register at [Amadeus for Developers](https://developers.amadeus.com)
2. Create a new app
3. Copy Client ID and Client Secret
4. Use the **Test environment** URLs (included in workflow)

### 3. Import Workflow to n8n
1. Copy the JSON content from `amadeus_tickets.json`
2. In n8n, go to: **Workflows** → **Import from File** → paste JSON
3. Configure credentials for Telegram

### 4. Update Configuration
Edit the **Konfiguracja** node and replace these values:

```javascript
const CONFIG = {
  maxFlightPrice: 700,              // Maximum price in PLN
  maxHotelPricePerNight: 200,       // For future use
  maxMotorcyclePricePerDay: 50,     // For future use
  tripDurationDays: 4,              // Trip length
  searchFromDays: 2,                // Start search from X days ahead
  searchToDays: 7,                  // Search until X days ahead
  telegramChatId: 'YOUR_CHAT_ID',   // 👈 Replace with your Chat ID
  amadeusClientId: 'YOUR_CLIENT_ID', // 👈 Replace with your Client ID
  amadeusClientSecret: 'YOUR_SECRET', // 👈 Replace with your Secret
  rapidApiKey: 'YOUR_RAPIDAPI_KEY'  // For future features
};
```

### 5. Add More Destinations (Optional)
Uncomment or add more airports and destinations in the configuration:

```javascript
const polishAirports = [
  {code: 'WRO', city: 'Wrocław'},
  {code: 'WAW', city: 'Warsaw'},
  {code: 'KRK', city: 'Krakow'},
  {code: 'GDN', city: 'Gdansk'},
  // Add more...
];

const islandDestinations = [
  {code: 'HER', name: 'Crete', city: 'Heraklion'},
  {code: 'PMI', name: 'Majorca', city: 'Palma'},
  {code: 'IBZ', name: 'Ibiza', city: 'Ibiza'},
  // Add more...
];
```

## 🎯 How It Works

1. **Configuration**: Generates all flight combinations (origin × destination × dates)
2. **Authentication**: Gets OAuth token from Amadeus API
3. **Rate Limiting**: 15-second delay between requests
4. **Flight Search**: Queries Amadeus Flight Offers API
5. **Price Analysis**: Compares prices against your limits
6. **Categorization**:
   - 🚀 **Super Deal**: < 70% of max price (< 490 PLN)
   - ✅ **Good Deal**: ≤ max price (≤ 700 PLN)
   - ❌ **Too Expensive**: > max price
7. **Notification**: Sends formatted message to Telegram

## 📱 Notification Examples

### Super Deal Alert
```
🚀 SUPER DEAL! 🚀

✈️ FLIGHT: Wrocław ➡️ Crete
💰 Price: 420 PLN (98 EUR)
📅 Departure: 15.03.2024
📙 Return: 19.03.2024
🕐 Departure time: 14:30
🕐 Arrival time: 18:45
✈️ Airline: W6
⏱️ Flight time: PT4H15M
🔄 Stops: 0

💎 You save: 280 PLN (40%)

⚡ BOOK IMMEDIATELY!

#CheapFlights #SuperDeal #Crete
```

### No Attractive Deals
```
❌ NO ATTRACTIVE DEALS ❌

✈️ Route: Wrocław ➡️ Malta
📅 Departure: 20.03.2024
📙 Return: 24.03.2024
💰 Cheapest flight: 850 PLN
🎯 Your limit: 700 PLN
📈 Too expensive by: 150 PLN

#CheapFlights #NoDeals
```

## 🔧 Customization Options

### Price Limits
- Adjust `maxFlightPrice` for different budgets
- Currency conversion (EUR → PLN) is automatic

### Search Dates
- `searchFromDays`: How many days ahead to start searching
- `searchToDays`: How many days ahead to search until
- `tripDurationDays`: Length of trip

### Deal Categories
Modify thresholds in the **Process Flights** node:
```javascript
isSuperDeal: priceInPln < (maxPrice * 0.7), // 70% of max price
isGoodDeal: priceInPln <= maxPrice,          // Within budget
```

## 🔄 Automation Setup

### Manual Execution
Click "Execute workflow" button in n8n

### Scheduled Execution
Add a **Cron** or **Schedule Trigger** node:
- Replace "Manual Trigger" with "Cron"
- Set schedule (e.g., daily at 9:00 AM)
- Connect to "Konfiguracja" node

### Webhook Trigger
For external triggering via HTTP requests

## ⚠️ Important Notes

### API Limits
- **Amadeus Test Environment**: Limited requests per month
- **Rate Limiting**: 15-second delays prevent API abuse
- **Batch Processing**: Processes one route combination at a time

### Production Usage
- For production, change API URL to: `https://api.amadeus.com`
- Consider upgrading to paid Amadeus plan for higher limits
- Test thoroughly with small batches first

### Data Privacy
- API keys are stored in n8n credentials
- Chat IDs are included in workflow data
- No flight booking data is stored

## 🐛 Troubleshooting

### Common Issues

**"Access token invalid"**
- Check Amadeus API credentials
- Verify you're using test environment URLs
- Ensure Client ID/Secret are correct

**"No flights found"**
- Check airport codes (IATA format: WAW, WRO, etc.)
- Verify date format (YYYY-MM-DD)
- Some routes might not be available

**Telegram not working**
- Verify bot token in n8n credentials
- Check chat ID format (usually a number)
- Ensure bot can send messages to your chat

**Rate limiting errors**
- Increase wait time between requests
- Reduce number of route combinations
- Check Amadeus API quotas

### Debug Mode
Enable debug logging in **Process Flights** node to see:
- API responses
- Price calculations
- Deal classifications

## 🔮 Future Enhancements

### Planned Features
- 🏨 Hotel price monitoring (RapidAPI integration ready)
- 🏍️ Motorcycle rental deals
- 📊 Price history tracking
- 🗺️ Multi-city trip support
- 📈 Price trend analysis

### Possible Extensions
- Email notifications
- Discord/Slack integration
- Web dashboard
- Mobile app notifications
- Price prediction ML model

## 📄 License

This workflow is provided as-is for educational and personal use. Please respect API terms of service and rate limits.

## 🤝 Contributing

Feel free to:
- Add more destinations
- Improve message formatting
- Add error handling
- Share your modifications

## 📞 Support

For issues with:
- **n8n**: Check [n8n documentation](https://docs.n8n.io)
- **Amadeus API**: Visit [Amadeus developers](https://developers.amadeus.com)
- **Telegram Bot**: Check [Telegram Bot API](https://core.telegram.org/bots/api)

---

**⚡ Happy deal hunting! ⚡**
