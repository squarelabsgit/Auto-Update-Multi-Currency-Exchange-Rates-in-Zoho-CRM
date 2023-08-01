# Auto-Update-Multi-Currency-Exchange-Rates-in-Zoho-CRM
Blog Post: https://www.squarelabs.com.au/post/auto-update-multi-currency-exchange-rates-in-zoho-crm

YouTube: https://youtu.be/YsmFMfrSVOw

If you're a Zoho CRM administrator looking to simplify and expedite the process of updating your currency exchange rates, you've come to the right place.

Zoho CRM's Multi-Currency feature is a powerful tool for businesses dealing with multiple currencies. However, one of its known limitations is the need for manual updates of the default exchange rate in the Settings under Company Details.

Manually updating these rates in your CRM system can be time-consuming and prone to errors. That's why we're here to show you how to automate this process by creating a custom schedule in Zoho CRM.

Our comprehensive tutorial will walk you through the entire setup, empowering you to automatically fetch the latest currency exchange rates effortlessly and allow you to say goodbye to the tedious task of manual updates so you can enjoy the benefits of accurate and up-to-date rates within your CRM.

## External API
Zoho doesn't have built-in currency updating tools, so we need to obtain exchange rates from an External API. There are multiple currency APIs that offer this service, but the one I found to be the simplest is the Exchange Rate API. They provide a free Open API to get exchange rates that update daily and don't require an API Key or setting up an account. However, you can create a free account to increase the number of calls to the service. For this project, though, there is no need to do so.

Their API is straightforward to use, as it only requires a simple GET request with our home currency at the end:

https://open.er-api.com/v6/latest/ + homeCurrencyISOCode

The data we receive from this API shows the exchange rates with our home currency in a format that is easy to use and push back into Zoho CRM.

## Connection
As we cannot update the currencies using standard Zoho Deluge integration tasks, we need to use the External Zoho API and call it from within our function. Zoho allows us to create connections for pre-determined scopes, which takes the difficulty out of having to manage authorisation and refresh tokens.

1. Head to Settings > Developer Space > Connections
2. Click Create Connection
3. Select Zoho OAuth
4. Enter a Connection Name e.g. "currency_connection"
5. Select the required scopes:
   * ZohoCRM.settings.currencies.UPDATE
   * *ZohoCRM.settings.currencies.READ
7. Click Create and Connect
8. Click Connect
9. Select your Production System
10. Click Accept

## Function
Of course we need a function to do all the heavy lifting, It will need to:
Get the current active system currencies and home currency
Get the exchange rates based on the home currency
Update the active currencies in our system
Head to Settings > Developer Space > Functions
Click New Function
Enter a Function Name (No Spaces)
Enter a Display Name
Set the Category to Schedule
Click Create
Enter code below
Click Save

Auto Update Currency Deluge Code
```js
//Required Variables
orgCurrencyData = Map();
orgCurrencyISOList = List();
updateCurrencyList = List();
//Get active currencies and home currency
getCurrencies = invokeurl
[
	url :"https://www.zohoapis.com/crm/v4/org/currencies"
	type :GET
	connection:"currency_connection"
];
orgCurrencies = getCurrencies.get("currencies").toList();
//Go through organisation currencies and get Base Currency ID
for each  currency in orgCurrencies
{
	//Getting the Base Currency ISO Code
	//Creating a list of active currency ISO codes to get exchange rates for
	//Storing the currency data by ISO Code for easy reference
	if(currency.get("is_base"))
	{
		baseCurrencyISOCode = currency.get("iso_code");
	}
	else if(!currency.get("is_base") && currency.get("is_active"))
	{
		orgCurrencyISOList.add(currency.get("iso_code"));
		orgCurrencyData.put(currency.get("iso_code"),currency);
	}
}
//API request to get a list of exchange rates based on the home currency ISO Code.
getBaseCurrencyRates = invokeurl
[
	url :"https://open.er-api.com/v6/latest/" + baseCurrencyISOCode
	type :GET
];
//Ensure API response was a success
if(getBaseCurrencyRates.get("result") == "success")
{
	baseCurrencyRates = getBaseCurrencyRates.get("rates");
	//loop through org currencys ISO values
	for each iso in orgCurrencyISOList
	{
		//add the required org currency ID to a list with the exchange rate.
		updateCurrencyList.add({"id":orgCurrencyData.get(iso).get("id"),"exchange_rate":baseCurrencyRates.get(iso).round(9).toString()});
	}
	info updateCurrencyList;
	//Prepare Param Map and pass in the list of currencies to update.
	paramMap = Map();
	paramMap.put("currencies",updateCurrencyList);
  //Update currencies in our system
	putCurrencies = invokeurl
	[
		url :"https://www.zohoapis.com/crm/v4/org/currencies"
		type :PUT
		parameters:paramMap.toString()
		connection:"currency_connection"
	];
	info putCurrencies;
}
```

## Schedule
Now that we have our function saved we need to attach it to a schedule.

Head to Settings > Automation > Schedules
Click Create Schedule
Enter a Schedule Name
Select Function to be executed > From existing functions
Click configure next to the function you just created
Click Save
Select a Start Date and Time
Set frequency to Daily
Set Ends to Never
Click Save
Click Run Now under Next Run Column
You have now updated your currencies using the Exchange Rate API

Make sure to subscribe to our channel for more valuable tutorials and tips on optimising your Zoho CRM experience.

## Resources:
Rates by Exchange Rate API: https://www.exchangerate-api.com

API documentation: https://www.exchangerate-api.com/docs/free

Available Currencies: https://www.exchangerate-api.com/docs/supported-currencies

GitHub Code: https://github.com/squarelabsgit/Auto-Update-Multi-Currency-Exchange-Rates-in-Zoho-CRM

Zoho API Documentation: https://www.zoho.com/crm/developer/docs/api/v4/get-currencies-data.html

<a href="http://www.youtube.com/watch?feature=player_embedded&v=YsmFMfrSVOw" target="_blank"><img src="http://img.youtube.com/vi/YsmFMfrSVOw/0.jpg" 
alt="YouTube Video" width="240" height="180" border="10" /></a>
