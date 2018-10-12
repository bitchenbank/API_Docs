## API General Convention
* All timestamps are accurate to milliseconds, in a 13-bit bigint format
* Public parameters when the HTTP request: lang, packtype, version, usertoken
* The content returned from each HTTP request is a json-formatted string
* If the data is requested by post, the post data should be in json format as far as possible, and the json string should be encrypted and placed in the param field of the post form


**Parameters:**

Parameter | HTTP method | Encrypted | is Required | Type | Meaning
------------ | ------------ | ------------ | ------------ | ------------ | ------------
usertoken | GET | no | yes | string | current user's token
packtype | GET | no | yes | int | PcWeb=1 H5=2 IOS=3 Android=4
version | GET | no | yes | string | 
lang | GET | no | yes | string | en represents English and cn represents Chinese
operatingsystem | POST, encrypt json-formatted string | yes | no | string | device operating system
deviceid | POST, encrypt json-formatted string | yes | no | string | device identification
**apikey** | POST,  encrypt json-formatted string | yes | no | string | when there is an apikey in the request and the length is greater than zero, the server will verify the apikey and not the usertoken


**lang**

code | parameters
------------ | ------------
cn | simplified Chinese
de | German
en | English
es | Spain
fr | French
ko | Korean
jp | Japanese
ru | Russian
tw | traditional Chinese


An example of requesting data
```
curl -X GET 'https://hq.bitchen.io/v1/klines.ashx?lang=cn&packtype=1&version=9.9.9&usertoken=aaaaaaaa&symbol=YOYO_BTC&interval=1m'
```
```
curl -X POST 'https://broker.bitchen.io/queryorder.ashx?lang=cn&packtype=1&version=9.9.9&usertoken=aaaaaaaaa' -d 'param=DA5D47C0F8762663CB24E9252D4B033428FFF5407AFB87C2ADD86624E3CF1FD094EEF19BC165A6DDA385066F43D860CC56DA5B7959982B4332E91C9E98B1A1EF6647AF4299BB79C00CEBAC46CAC20F2F57AB3C96CADA50237C67F3728A5753E4'
```

* the string after the param parameter is the encrypted json parameter
* response unifies the three json fields (The resulting content returned from each HTTP request is a json-formatted string): status, message and data, and puts the content of each API in the data field


**status 状态：**
* 1 indicates success
* -1 fail, message field displays the error message
* -2 needs to login 
* -3 requires login
* -4 if the balance is insufficient, need to pop up the recharge message box
* -5 password retry too many, need to pop up message box to retrieve password 
message: error messages, which display error messages when status == -1




## Post encryption

* Encryption method：3des
* encryption mode：**CBC** 
* Padding：**pkcs7padding** 
* Output： **hex**

desKey and desIV: Displayed on the apikey setting page on bitchen.io (https://web.bitchen.io/apikeys)



**C# demo**
```C#

public static string EncodeParam(string param, string desKey, string desIV)
{
    byte[] data = Encoding.UTF8.GetBytes(param);
    using (TripleDESCryptoServiceProvider tdes = new TripleDESCryptoServiceProvider())
    {
        tdes.Mode = CipherMode.CBC;
        tdes.Padding = PaddingMode.PKCS7;
        tdes.Key = Encoding.UTF8.GetBytes(desKey);
        tdes.IV = Encoding.UTF8.GetBytes(desIV);
        ICryptoTransform cTransform = tdes.CreateEncryptor();
        byte[] resultArray = cTransform.TransformFinalBlock(data, 0, data.Length);
        SoapHexBinary shb = new SoapHexBinary(resultArray);
        return shb.ToString();
    }
}

public static string DecodeParam(string hexString, string desKey, string desIV)
{
    SoapHexBinary shb = SoapHexBinary.Parse(hexString);
    byte[] data = shb.Value;
    using (TripleDESCryptoServiceProvider tdes = new TripleDESCryptoServiceProvider())
    {
        tdes.Key = Encoding.UTF8.GetBytes(desKey);
        tdes.IV = Encoding.UTF8.GetBytes(desIV);
        ICryptoTransform cTransform = tdes.CreateDecryptor();
        byte[] resultArray = cTransform.TransformFinalBlock(data, 0, data.Length);
        string ret = UTF8Encoding.UTF8.GetString(resultArray);
        return ret;
    }
}

```

**PHP demo**
```PHP

<?php
class CryptDes {
     var $key;
     var $iv;
     function CryptDes($key, $iv){
        $this->key = $key;
        $this->iv = $iv;
     }
                  
     function encrypt($input){
         $size = mcrypt_get_block_size(MCRYPT_3DES,MCRYPT_MODE_CBC);
         $input = $this->PaddingPKCS7($input);
         $td = mcrypt_module_open(MCRYPT_3DES,'',MCRYPT_MODE_CBC, '');        
         @mcrypt_generic_init($td, $this->key, $this->iv);
         $data = mcrypt_generic($td, $input);
         mcrypt_generic_deinit($td);
         mcrypt_module_close($td);
         return strtoupper(bin2hex($data));
     }
              
     function decrypt($encrypted){        
         $encrypted = $this->hex2bin($encrypted);       
         $td = mcrypt_module_open(MCRYPT_3DES,'',MCRYPT_MODE_CBC,'');         
         $ks = mcrypt_enc_get_key_size($td);
         @mcrypt_generic_init($td,  $this->key, $this->iv);
         $decrypted = mdecrypt_generic($td, $encrypted);
         mcrypt_generic_deinit($td);
         mcrypt_module_close($td);
         $y=$this->UnPaddingPKCS7($decrypted);
         return $y;
     }
              
     function hex2bin($hexData)
     {
        $binData = "";
        for($i = 0; $i < strlen ($hexData); $i += 2)
        {
           $binData .= chr(hexdec(substr($hexData, $i, 2)));
        }        
        return $binData;
    }
              
     function PaddingPKCS7($data) {
         $block_size = mcrypt_get_block_size(MCRYPT_DES, MCRYPT_MODE_CBC);
         $padding_char = $block_size - (strlen($data) % $block_size);
         $data .= str_repeat(chr($padding_char),$padding_char);
         return $data;
     }

     private function UnPaddingPKCS7 ($text)
     {
        $pad = ord($text{strlen($text) - 1});
        if ($pad > strlen($text)) {
            return false;
        }
        if (strspn($text, chr($pad), strlen($text) - $pad) != $pad) {
            return false;
        }
        return substr($text, 0, - 1 * $pad);
     }
}
              
$des = new CryptDes("key","iv");//（secret key vector, obfuscation vector）
echo $ret = $des->decrypt("test");//encrypted string
?>

```


# Market Data API

## Get the product list


```
GET https://hq.bitchen.io/v1/product.ashx?curMarket={curMarket}&assetCode={assetCode}&lang={lang}&version={version}
```

* curMarket: market classification, if not transmitted then return all market data
* assetCode: asset currency, if not transmitted then return all market data
* lang: language
* version: version



**Payload:**
```javascript
{	
    "status": "1",
	"message": "success",
	"data": [{
		"symbol": "ETH_BTC", //symbol
		"status": true, //opening status
		"statusText": "", //opening description
		"name": "ETH/BTC", //name of the symbol
		"assetCode": "ETH", //asset
		"curMarket": "BTC", //currency
		"tickSize": "0.000000", //minimum price fluctuation
		"decimalplace": "6", //decimalplace
		"lots": "6", //minimum number of transactions
		"minOrderAmount": "6", //minimum transaction amount
		"open": "0.000000", //24 hours opening price
		"high": "0.000000", //24 hours highest price
		"low": "0.000000", //24 hours lowest price
		"close": "0.000000", //the latest price
		"volume": "0.000000" //trading volume
		"upDown": "0.000000" //change amount
		"upDownRate": "1%" //change rate
		"tradedMoney": "12345.90" // value of traded money
		"tickerid": 111 //ticker id
		"legalAmount": “$100” //legal tender amount
	}]
}
```


## Obtaining trades data

```
GET https://hq.bitchen.io/v1/aggtrades.ashx?symbol={symbol}&limit={limit}&lang={lang}&version={version}
```

* limit: number of trades
* lang: language
* version: version



**Payload:**
```javascript
{
    "status": "1",
    "message": "success",
    "data": [{
        "a": "26129", // Aggregate tradeId
        "p": "0.01633102", // Price
        "q": "4.70443515", // Quantity
        "f": "27781", // First tradeId
        "l": "27781", // Last tradeId
        "T": "1498793709153", // Timestamp
        "m": true, // Was the buyer the maker?
        "M": true // Was the trade the best price match?
    }]
}
```

## Obtaining orderbook
```
GET https://hq.bitchen.io/v1/depth.ashx?symbol={symbol}&limit={limit}&lang={lang}&version={version}
```

* limit: depth
* lang: language
* version: version



**Payload:**
```javascript
{
  "status": "1",
  "message": "success",
  "data": {
		"symbol": "ETH_BTC", //symbol
		"name": "ETH/BTC",  //name of the symbol
		"assetCode": "ETH",  //asset
		"curMarket": "BTC",  //currency
		"lastupdateid": "1",  //the last updated orderbookid
		"risefall": "0",  //price color, 0 white 1 red 2 green
		"lastPrice": "8",  //the latest price
		"decimalplace": "8",  //decimalplace
		"bidsFilter": "1",  //depth graph rule, the max value of bids
		"asksFilter": "1",  //depth graph rule, the min value of asks
		"asksBgRule": "1",  //background color ASKs 
		"bidsBgRule": "1",  //background color BIDs
		"bids": [{price: "0.092000", quantity: "1.000", amount: "0.09200000"},{price: "0.092000", quantity: "1.000", amount: "0.09200000"}],    //bid
		"asks": [{price: "0.092000", quantity: "1.000", amount: "0.09200000"}]       //ask
  }
}
```



## Get klines API
```
GET https://hq.bitchen.io/v1/klines.ashx?symbol={symbol}&interval={interval}&lang={lang}&version={version}
```

1. interval:
			time , 1m , 3m , 5m , 15m , 30m , 1h , 2h , 4h , 6h , 8h , 12h , 1d , 1w

2. lang: language
3. version: version
4. starttime: long, starttime
5. endtime: long, endtime
6. count: amount of the record, default 3000



**Payload:**
```javascript
{
	"status": "1",
	"message": "success",
	"data": {
		"symbol": "ETH_BTC", //symbol
		"status": true, //opening status
		"statusText": "", //opening description
		"name": "ETH/BTC", //name of the symbol
		"assetCode": "ETH", //asset
		"curMarket": "BTC", //currency
		"tickSize": "0.000000", //minimum price fluctuation
		"decimalplace": "6", //decimalplace
		"open": "0.000000", //24 hours opening price
		"high": "0.000000", //24 hours highest price
		"low": "0.000000", //24 hours lowest price
		"close": "0.000000", //the latest price
		"volume": "0.000000", /trading volume
		"upDown": "0.000000", //change amount
		"upDownRate": "1%", //change rate
		"tradedMoney": "12345.90", //value of traded money
		"lastTime": "1516125780000", //the latest transaction time
		"lastTimeDataTime": "1516125780000", //time of the latest timeline, used for webscoket checking
		"legalAmount": "￥100" //legal tender amount
		"timeData": [
			{
			"time": "1516125780000", //time
			"open": "0.00000028", //opening price
			"low": "0.00000001", //lowest price
			"high": "0.000001", //highest price
			"close": "0.00000023", //closing price
			"vol": "0.00332783" //volume
			"val": "0.00332783" //value
			"prevClose": "0.00332783" //closing price of last 24h
			"upDown": "0.000000" //change amount
			"upDownRate": "1%" //change rate
			}
		            ]
	        }
}
```



# transaction API

## buypage API
```
GET https://broker.bitchen.io/buypage.ashx?&packtype={packtype}&version={version}
```

* param is encrypted from the incoming parameters in the following ways:

**Parameters:**

{"usertoken":"test","symbol":"ETH_BTC"}

**Payload:**
```javascript
{
	"status": "1",
	"message": "",
	"data": {
		"assetcode": "asset type",
		"curmarket": "currency market",
		"assetcamount": "assetcode balance (sellable)",
		"curamount": "curmarket balance（buyable）",
		"tradefeerate": "trade fee rate",
		"hasdiscount": "whether there is a discount on the commission rate",
		"tradefeediscount": "the percentage of fees charged after a discount",
		"sellmaxvol": "maximum sellable quantity",
		"minpx": "the smallest unit of the order price",
		"minvol": "minimum volume of transactions",
		"minval": "minimum transaction amount",
		"morderpxrange": "the floating ratio of the price of the market order based on the latest price",
		"minvalplace": "minimum transaction unit",
		"legalrate": "ratio of legal tender's valuation",
		"legalcur": "unit of legal tender"
	}
}
```



## order API
```
GET https://broker.bitchen.io/order.ashx?&packtype={packtype}&version={version}
```

* param is encrypted from the incoming parameters in the following ways:

**Parameters:**

{"usertoken":"aaa","assetcode":"GAS","curmarket":"BTC",...}


parameter  | HTTP method | encrypts | is Required | type | meaning
------------ | ------------ | ------------ | ------------ | ------------ | ------------
usertoken | POST | yes | yes | string | user's token
symbol |  | yes |   |  string | symbol
assetcode |  | yes |  |  string | asset type
curmarket |  | yes |  | string | currency market
ordertype |  | yes |  | int	 | order type. Market order 1; Price limit order 2; Stop-Limit 3
bstype |  | yes |  | int  | buy and sell. Buy 1; Sell 2 
price |  | yes |  | decimal | order price
quantity |  | yes |  | decimal | order quantity
triggerdir |  | selectable |  | int |  | larger than the trigger price on the trigger direction 1; Less than or equal to the trigger price on the trigger direction 2
triggerprice |  | selectable |  | decimal | trigger prices
packtype |  | yes |  | int |  | packtype | package type
devicetype |  | no |  | string |  | device model (huawei, xiaomi, iphone7, chrome, Safari, etc.)
deviceid |  | no |  | string |  | device unique identification ID
timewindow |  | no |  | bigint |  | timewindow. The default is 5000, per ms
timeinforce |  | no |  | string |  | order expiration time limit. IOC；GTC

  

**Payload:**
```javascript
{
	"status": "1",
	"message": "order request sent",
	"data": {}
}

{
	"status": "-1",
	"message": "failed to place order",
	"data": {}
}

{
	"status": "4",
	"message": "insufficient balance ",
	"data": {}
}
```



## execute report API
```
GET https://broker.bitchen.io/orderexedetail.ashx?&packtype={packtype}&version={version}
```


**Parameters:**

{"usertoken":"aaa","orderno":"1"}


**Payload:**
```javascript
{
	"status": "1",
	"message": "",
	"data": {
		"totalfee": "total trade fee",
		"totalamount": "gross filled amount",
		"list": [{
			"exetime": "trade time",
			"filledprice": "strike price",
			"filledquantity": "filled quantity",
			"tradefee": "trade fee",
			"filledamount": "filled amount"
		}]
	}
}
```



## Query the current order API
```
GET https://broker.bitchen.io/queryorder.ashx?&packtype={packtype}&version={version}
```


**Parameters:**

{"usertoken":"test"}


**Payload:**
```javascript
{
	"status": "1",
	"message": "",
	"data": {
		"list": [{
			"orderno": "order ID (used when withdrawing the order)",
			"ordertime": " order time",
			"symbol": "symbol",
			"name": "name",
			"ordertype": "order type",
			"ordertypename": "The description of ordertype",
			"orderstatus": "order status",
			"bstype": "buy or sell type",
			"bstypename": "The literal form of bstype",
			"orderprice": "order price",
			"orderquantity": "order quantity",
			"filledquantity": "filled quantity",
			"filledrate": "filled rate",
			"amount": "amount",
			"triggercon": "trigger condition",
			"bsandotype": "bsandotype"
		}]
	}
}
```




## User funds list API
```
GET https://broker.bitchen.io/userasset.ashx?&packtype={packtype}&version={version}
```


**Parameters:**

{"usertoken":"test","sortfield":"amount","sortdir":"1"}


**Payload:**
```javascript
{
	"status": "1",
	"message": "",
	"data": {
		"totalval": "current total valuation",
		"btctotalval": "btc valuation",
		"legaltotalval": "legal valuation",
		"withdrawlimit": "24 hours withdraw limit",
		"limitused": "limit used",
		"pettyval": "value of defining a small asset",
		"list": [{
			"assetcode": "asset",
			"assetfullname": "asset full name",
			"amount": "total amount",
			"canusedamount": "available balance",
			"frozenamount": "frozen transaction order amount",
			"frozenwithdraw": "frozen withdral amount",
			"btcval": "btc valuation",
			"legalval": "legal valuation",
			"depositstatus": "deposit button status",
			"withdrawstaus": "withdraw button status",
			"tradestatus": "trade button status",
			"hastag": " ",
			"logourl": "image URL"
		}]
	}
}
```






