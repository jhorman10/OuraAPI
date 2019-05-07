/*
Oura API 
*/
var CLIENT_ID = '...';
var CLIENT_SECRET = '...';

/**
 * Authorizes and makes a request to the Oura API.
 */
 
function run() {
  var service = getService();
  if (service.hasAccess()) {
    var token = service.getAccessToken();
    var urlUser = 'https://api.ouraring.com/v1/userinfo?access_token='+token+'.json';
    var urlActivity = 'https://api.ouraring.com/v1/activity?=start2019-01-01&end=actually?access_token='+token+'.json';
    var urlSleep = 'https://api.ouraring.com/v1/sleep?start=2019-01-01&end=actually?access_token='+token+'.json';
    var urlReadiness = 'https://api.ouraring.com/v1/readiness?=start2019-01-01&end=actually?access_token='+token+'.json';
    
    // JSON_USER
    var responseUser = UrlFetchApp.fetch(urlUser, {
      headers: {
        Authorization: 'Bearer ' + token
      }
    });
    var result = JSON.parse(responseUser.getContentText());
    Logger.log(result);
    SpreadsheetApp.getActiveSpreadsheet().getSheetByName('User').getRange('A1').setValue(JSON.parse(responseUser.getContentText()));    
    
    //JSON_ACTIVITY
    var responseActivity = UrlFetchApp.fetch(urlActivity, {
      headers: {
        Authorization: 'Bearer ' + token
      }
    });
    var result = JSON.parse(responseActivity.getContentText());
    Logger.log(result);
    SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Activity').getRange('A1').setValue(responseActivity); 
    
    //JSON_SLEEP
    var responseSleep = UrlFetchApp.fetch(urlSleep, {
      headers: {
        Authorization: 'Bearer ' + token
      }
    });
    var result = JSON.parse(responseSleep.getContentText());
    Logger.log(result);
    SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Sleep').getRange('A1').setValue(responseSleep);    
    
    
    //JSON_READINESS
    var responseReadiness = UrlFetchApp.fetch(urlReadiness, {
      headers: {
        Authorization: 'Bearer ' + token
      }
    });
    var result = JSON.parse(responseReadiness.getContentText());
    Logger.log(result);
    SpreadsheetApp.getActiveSpreadsheet().getSheetByName('Readiness').getRange('A1').setValue(responseReadiness);   
    
  } else {
    var JSON_USER = service.getAuthorizationUrl();
    Logger.log('Open the following URL and re-run the script: %s',
        JSON_USER);
  }
}

/**
 * Reset the authorization state, so that it can be re-tested.
 */
function reset() {
  getService().reset();
}

/**
 * Configures the service.
 */
function getService() {
  return OAuth2.createService('Oura')
      // Set the endpoint URLs.
      .setAuthorizationBaseUrl('https://cloud.ouraring.com/oauth/authorize')
      .setTokenUrl('https://api.ouraring.com/oauth/token')

      // Set the client ID and secret.
      .setClientId(CLIENT_ID)
      .setClientSecret(CLIENT_SECRET)

      // Set the name of the callback function that should be invoked to
      // complete the OAuth flow.
      .setCallbackFunction('authCallback')

      // Set the property store where authorized tokens should be persisted.
      .setPropertyStore(PropertiesService.getUserProperties())

      // Set the scope and additional headers required by the FitBit API.
      .setScope('personal email daily')
      //.setTokenHeaders({
        //'Authorization': 'Basic ' +
          //  Utilities.base64Encode(CLIENT_ID + ':' + CLIENT_SECRET)
      //});
}


function showSidebar() {
  var ouraService = getService();
  if (!ouraService.hasAccess()) {
    var authorizationUrl = ouraService.getAuthorizationUrl();
    var template = HtmlService.createTemplate(
        '<a href="<?= authorizationUrl ?>" target="_blank">Authorize</a>. ' +
        'Reopen the sidebar when the authorization is complete.');
    template.authorizationUrl = authorizationUrl;
    Logger.log(authorizationUrl);
    var page = template.evaluate();
    SpreadsheetApp.getUi().showSidebar(page);
  } else {
    //
  }
}
/**
 * Handles the OAuth callback.
 */
function authCallback(request) {
  var service = getService();
  var authorized = service.handleCallback(request);
  if (authorized) {
    return HtmlService.createHtmlOutput('Success!');
  } else {
    return HtmlService.createHtmlOutput('Denied.');
  }
}

/**
 * Logs the redict URI to register.
 */
function logRedirectUri() {
  Logger.log(OAuth2.getRedirectUri());
}

