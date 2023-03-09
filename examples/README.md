Examples
========
**TODO**: Replace with directory structure and examples from exsisting libraries as much as possible.

## Entities / Objects as Arguments
    //based off the current java client library
    TextMessage message = new TextMessage(SMS_FROM, SMS_TO, SMS_TEXT);
    results = client.sms.send(message);
    
## Entities / Objects Reuse
    //php style psudocode
    $request = new VerifyRequest(NUMBER, BRAND);
    $request->setCodeLength(6);
      
    $verification = $client->verify->start($request);
    $status = $client->verify->status($verification);
      
    //put $verification into memcache, pull out on another request
      
    $result = $client->verify->check($verification, CODE);
    
## Authorization 
    //from php library
    use Vonage\Client as VonageClient;
    use Vonage\Credentials\Basic;
    use Vonage\Credentials\OAuth;
      
    $client = new VonageClient(new Basic(API_KEY, API_SECRET));

## Optional Settings
    //php style psudocode, proxy all requests through runscope
    $auth = new Basic(API_KEY, API_SECRET);
    $client = new VonageClient($auth, ['https://api-vonage-com-6r3o7ueg4emv.runscope.net/', 'https://rest-vonage-com-6r3o7ueg4emv.runscope.net/'], SHARED_SECRET);
    
## Rate Limited APIs
    //php style psudocode, default rate handling
    $message = new TextMessage(TO, FROM, TEXT);
    foreach($numbers as $number){
        $message->setTo($number);
        //no exception thrown means all is okay
        $response = $vonageClient->sms->send($message);
        //if rate limit response is received, client automatically backs off and tries again
    }
      
    //disable automatic rate limit protection
    $response = $vonageClient->sms->send($message, false);
      
    //exception thrown if account not setup for queueing, or hit queue limit
    $response = $vonageClient->sms->send($message, false);
      
    //set optimistic rate limit of 10 sms / second (avoid errors, back off automatically)
    foreach($numbers as $number){
        $message->setTo($number);
        //no exception thrown means all is okay
        $response = $vonageClient->sms->send($message, 10);
        //if rate limit response is received, client automatically backs off and tries again
    }
    
## Callback Support
    //from php library
    try{
        $callback = Vonage\Network\Number\Callback::fromEnv();
    } catch (Exception $e) {
        error_log('not a valid NI callback: ' . $e->getMessage());
        return;
    }
     
    if($callback->hasType()){
        echo $callback->getNumber() . ' is a ' . $callback->getType() . ' number';
    }

## Callback / Entity Support: Adding Additional Data
    //from php library
    //fetch partial response from memcache
    $response = $memcached->get($callback->id());
     
    //this will create a new response object with both the API response data, and the callback data (appending the
    //callback data if another callback has already been added to the response)
    $response = Vonage\Network\Number\Request::addCallback($response, $callback);
     
    if($response->isComplete()){
        //store the data
    } else {
        $response = $memcached->set($callback->id(), $response);
    }

## Callback / Entity Support: Adding DLR
    //php style psudocode
    $message = $vonageClient->sms->send(new TextMessage(TO, TEXT, FROM));
      
    //in dlr callback
    $dlr = Vonage\Sms\DLR::fromEnv();
    $message = TextMessage::addDLR($dlr);
      
    //now message object contains DLR, can be put in key/value store for audit