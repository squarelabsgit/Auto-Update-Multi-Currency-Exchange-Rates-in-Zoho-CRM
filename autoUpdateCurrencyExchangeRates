//
//Refer to YouTube Video: https://youtu.be/YsmFMfrSVOw
//
//Required Variables
orgCurrencyData = Map();
orgCurrencyISOList = List();
updateCurrencyList = List();
//Get all organisations currencies
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
	//Creating a list of active organisation ISO currency codes to get exchange rates for
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
	for each  iso in orgCurrencyISOList
	{
		//add the required org currency ID to a list with the exchange rate.
		updateCurrencyList.add({"id":orgCurrencyData.get(iso).get("id"),"exchange_rate":baseCurrencyRates.get(iso).round(9).toString()});
	}
	info updateCurrencyList;
	//Prepare Param Map and pass in the list of currencies to update.
	paramMap = Map();
	paramMap.put("currencies",updateCurrencyList);
	putCurrencies = invokeurl
	[
		url :"https://www.zohoapis.com/crm/v4/org/currencies"
		type :PUT
		parameters:paramMap.toString()
		connection:"currency_connection"
	];
	info putCurrencies;
}
