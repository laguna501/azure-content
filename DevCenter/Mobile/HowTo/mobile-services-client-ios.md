<properties linkid="mobile-services-how-to-iOS-client" urlDisplayName="iOS Client Library" pageTitle="How to use the iOS client library - Windows Azure Mobile Services feature guide" metaKeywords="Windows Azure Mobile Services, Mobile Service iOS client library, iOS client library" writer="glenga" metaDescription="Learn how to use the iOS client library for Windows Azure Mobile Services." metaCanonical="" disqusComments="1" umbracoNaviHide="0" />

<div chunk="../chunks/article-left-menu-iOS.md"></div>

# How to use the iOS client library for Mobile Services
<div class="dev-center-tutorial-selector sublanding"> 
  <a href="/en-us/develop/mobile/how-to-guides/work-with-net-client-library/" title=".NET Framework">.NET Framework</a><a href="/en-us/develop/mobile/how-to-guides/work-with-html-js-client/" title="HTML/JavaScript">HTML/JavaScript</a><a href="/en-us/develop/mobile/how-to-guides/work-with-ios-client-library/" title="iOS" class="current">iOS</a><a href="/en-us/develop/mobile/how-to-guides/work-with-android-client-library/" title="Android">Android</a>
</div>

This guide shows you how to perform common scenarios using the iOS client for Windows Azure Mobile Services. The samples are written in objective-C and require the [Mobile Services SDK].  This tutorial also requires the [iOS SDK]. The scenarios covered include querying for data; inserting, updating, and deleting data; authenticating users; and handling errors. If you are new to Mobile Services, you should consider first completing the [Mobile Services quickstart][Get started with Mobile Services]. The quickstart tutorial helps you configure your account and create your first mobile service.

## Table of Contents

- [What is Mobile Services][]
- [Concepts][]
- [Setup and Prerequisites][]
- [How to: Create the Mobile Services client][]
- [How to: Create a table reference][]
- [How to: Query data from a mobile service][]
	- [Filter returned data]
    - [Using the MSQuery object][How to: Use MSQuery]
	- [Select specific columns]
- [How to: Insert data into a mobile service]
- [How to: Modify data in a mobile service]
- [How to: Bind data to the user interface]
- [How to: Authenticate users]
- [How to: Handle errors]

<!--- [How to: Design unit tests]
- [How to: Customize the client]
	- [Customize request headers]
	- [Customize data type serialization]
- [Next steps][]-->

<div chunk="../chunks/mobile-services-concepts.md" />

##<a name="Setup"></a>Setup and Prerequisites

This guide assumes that you have created a mobile service with a table.  For more information see [Create a table]. The examples in this topic use a table named `ToDoItem`, which has the following columns:

+ `id`
+ `text`
+ `complete`
+ `duration`

If you are creating your iOS application for the first time, make sure to add the `WindowsAzureMobileServices.framework` in your application's **Link Binary With Libraries** setting.

In addition, you must add the following reference in the appropriate files or in your application's .pch file.

	#import <WindowsAzureMobileServices/WindowsAzureMobileServices.h>

<h2><a name="create-client"></a><span class="short-header">Create the client</span>How to: Create the Mobile Services client</h2>

The following code creates the mobile service client object that is used to access your mobile service. 

	MSClient *client = [MSClient clientWithApplicationURLString:@"MobileServiceUrl" applicationKey:@"AppKey"]
	
In the code above, replace `MobileServiceUrl` and `AppKey` with the mobile service URL and application key of your mobile service. To determine these settings for your mobile service, in the Windows Azure Management Portal select your mobile service, then click **Dashboard**.

You can also create your client from an **NSURL** object that is the URL of the service, as follows:

	MSClient *client = [MSClient clientWithApplicationURL:(NSURL *)url
								 applicationKey:(NSString *)string];

<h2><a name="table-reference"></a><span class="short-header">Create table reference</span>How to: Create a table reference</h2>

Before you can access data from your mobile service, you must get a reference to the table from which you want to query, update, or delete items. In the following example, `ToDoItem` is the table name:

	MSTable *table = [client tableWithName:@"ToDoItem"]; 


<h2><a name="querying"></a><span class="short-header">Query data</span>How to: Query data from a mobile service</h2>

Once you have a MSTable object you can then create your query.  The following simple query gets all the items in our ToDoItem table.

	[table readWithCompletion:^(NSArray *items, NSInteger totalCount, NSError *error) {
		if(error) {
			NSLog(@"ERROR %@", error);
		} else {
			for(NSDictionary *item in items) {
				NSLog(@"Todo Item: %@", [item objectForKey:@"text"]);
			}
		}
	}];

Note that in this case we simply write the text of the task to the log.

The following parameters are supplied in the callback:

+ _items_: An **NSArray** of the records that matched your query.
+ _totalCount_: The total count of items in all pages of the query, not just those returned in the current page. This value is set to -1, unless you explicitly request the total count in your request. For more info, see [Return data in pages].
+ _error_: Any error that occurred; otherwise `nil`.

### <a name="filtering"></a>How to: Filter returned data

When you want to filter your results, you have a number of options available to you. 

The most common case is to use an NSPredicate to filter the results.

	[table readWithPredicate:(NSPredicate *)predicate completion:(MSReadQueryBlock)completion];
	
The following predicate returns only the incomplete items in our ToDoItem table:

	NSPredicate *predicate = [NSPredicate predicateWithFormat:@"complete == NO"];
	[table readWithPredicate:predicate completion:^(NSArray *items, NSInteger totalCount, NSError *error) {
		//loop through our results
	}];
	
A single record can be retrieved by using its Id.

	[table readWithId:[NSNumber numberWithInt:1] completion:^(NSDictionary *item, NSError *error) {
		//your code here
	}];

Note that in this case the callback parameters are slightly different.  Instead of getting an array of results and an optional count, you instead just get the one record back.

### <a name="query-object"></a>Using the MSQuery object

Use the **MSQuery** object when you need a query that is more complex than just filtering rows, such as changing the sort order on your results or limiting the number of data records you get back. The following two examples show how to create an MSQuery object instance:

+	`MSQuery *query = [table query];`	
+	`MSQuery *query = [table queryWithPredicate:(NSPredicate *)predicate];`

The MSQuery object enables you to control the following query behaviors:

* Specify the order results are returned.
* Limit which fields are returned.
* Limit how manu records are returned.
* Specify whether to include the total count in the response.
* Specify custom query string parameters in the request.

You further define a query by applying one or more functions. Once the query is defined, it is executed by calling the **readWithCompletion** function.

#### <a name="sorting"></a>Sorting returned data

The following functions are used to specify the fields used for sorting:

	-(void) orderByAscending:(NSString *)field
	-(void) orderByDescending:(NSString *)field
	
This query sorts the results first by duration and then by whether the task is complete:

	[query orderByAscending(@"duration")];
	[query orderByAscending(@"complete")];
	[query readWithCompletion:(NSArray *items, NSInteger totalCount, NSError *error) {
		//code to parse results here
	}];	

#### <a name="paging"></a>Returning data in pages 

Mobile Services limits the amount of records that are returned in a single response. To control the number of records displayed to your users you must implement a paging system.  Paging is performed by using the following three properties of the **MSQuery** object:

+	`BOOL includeTotalCount`
+	`NSInteger fetchLimit`
+	`NSInteger fetchOffset`

In the following example, a simple function requests 20 records from the server and then appends them to the local collection of previously loaded records:

	- (bool) loadResults() {
		MSQuery *query = [self.table query];

		query.includeTotalCount = YES;
		query.fetchLimit = 20;
		query.fetchOffset = self.loadedItems.count;
		[query readWithCompletion:(NSArray *items, NSInteger totalCount, NSError *error) {
			if(!error) {
				//add the items to our local copy
				[self.loadedItems addObjectsFromArray:items];		

				//set a flag to keep track if there are any additional records we need to load
				self.moreResults = (self.loadedItems.count < totalCount);
			}
		}];
	}

#### <a name="selecting"></a>Limiting the returned fields

To limit which field are returned from your query, simply specify the names of the fields you want in the **selectFields** property. The following example returns only the text and completed fields:

	query.selectFields = @["text", @"completed"];

#### <a name="parameters"></a>Specifying additional querystring parameters

The client library makes it possible to include additional querystring parameters in the request to the server. These parameters might be required by your server side scripts. The following example adds two querystring parameters to the request:   

	query.parameters = @{
		@"myKey1" : @"value1",
		@"myKey2" : @"value2",
	};

These parameters are appended to query URI as `myKey1=value1&myKey2=value2`. 
For more information, see [How to: access custom parameters].

<h2><a name="inserting"></a><span class="short-header">Inserting data</span>How to: Insert data into a mobile service</h2>

To insert a new row into the table, you create a new [NSDictionary object] and pass that to the insert function. The following code inserts a new todo item into the table:

	NSDictionary *newItem = @{"text": "my new item", @"complete" : @NO};
	[table insert:newItem completion:^(NSDictionary *result, NSError *error) {
		// The result contains the new item that was inserted,
		// depending on your server scripts it may have additional or modified 
		// data compared to what was passed to the server.
	}];	

<div class="dev-callout"><strong>Note</strong>
<p>When inserting an item, the insert fails when you manually set `id` in the dictionary object.</p>
</div>

When dynamic schema is enabled, Mobile Services automatically generates new columns based on the fields of the object in the insert or update request. For more information, see [Dynamic schema].

<h2><a name="modifying"></a><span class="short-header">Modifying data</span>How to: Modify data in a mobile service</h2>

Update an existing object by modifying an item returned from a previous query and then calling the **update** function.

	NSMutableDictionary *item = [self.results.item objectAtIndex:0];
	[item setObject:@YES forKey:"complete"];
	[table update:item completion:^(NSDictionary *item, NSError *error) {
		//handle errors or any additional logic as needed
	}];

When making updates, you only need to supply the field being update, along with the row ID, as in the following example:

	[table update:@{"id" : 1, "Complete": Yes} completion:^(NSDictionary *item, NSError *error) {
		//handle errors or any additional logic as needed
	}];
	
	
To delete an item from the table, simply pass the item to the delete method, as follows:

	[table delete:item completion:^(NSDictionary *item, NSError *error) {
		//handle errors or any additional logic as needed
	}];

You can also just delete a record using its id directly, as in the following example:

	[table deleteWithId:[NSNumber numberWithInt:1] completion:^(NSDictionary *item, NSError *error) {
		//handle errors or any additional logic as needed
	}];	

Note that, at minimum, the `id` attribute must be set when making updates and deletes.

<h2><a name="authentication"></a><span class="short-header">Authentication</span>How to: Authenticate users</h2>

Mobile Services enables you to use the following identity providers to authenticate users:

- Facebook
- Google 
- Microsoft Account
- Twitter

For more information about configuring an identity provider, see [Get started with authentication]. 

Mobile Services supports the following two authentication workflows:

- In a server-managed login, Mobile Services manages the login process on behalf of your app. A provider-specific login page is displayed by the client library, and Mobile Services does the work of authenticating with the chosen provider.

- In a client-managed login, the app must request a token from the identity provider and then present this token to Mobile Services for authentication.

When authentication succeeds, a user object is returned that contains the assigned user ID value and the authentication token. You can use this user ID in server scripts to validate or modify requests. For more information, see [Use scripts to authorize users]. The token itself can be securely cached to use in subsequent logins. 

You can also set permissions on tables to restrict access for specific operations to only authenticated users. For more information, see [Permissions].

### Server-managed login

Here is an example of how to login using Facebook. This code could be called in your controller's ViewDidLoad or manually triggered from a UIButton. This will display a standard UI for logging into the identity provider.

	[client loginWithProvider:@"MicrosoftAccount" controller:self animated:YES
		completion:^(MSUser *user, NSError *error) {
			NSString *msg;
			if(error) {
				msg = [@"An error occured: " stringByAppendingString:error.description];
			} else {
				msg = [@"You are logged in as " stringByAppendingString:user.userId];
			}
		
			UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Login" 
								  message:msg 
								  delegate:nil 
								  cancelButtonTitle:@"OK" 
								  otherButtonTitles: nil];
			[alert show];
	}];	

Note: if you are using an identity provider other than Facebook, change the value passed to the login method above to one of the following: microsoftaccount, twitter, or google.

You can also get a reference to the MSLoginController and display it yourself using:

	-(MSLoginController *)loginViewControllerWithProvider:(NSString *)provider completion:(MSClientLoginBlock)completion;

### Client-managed login (single sign-on)

There are cases when the login process is done outside of the Mobile Services client. You might do this to enable a single sign-on functionality or when your app must contact the identity provider directly to obtain user information. In these cases, you can login to Mobile Services by providing a token obtained independently from a supported identity provider.

The following example uses the [Live Connect SDK] to enable single sign-on for iOS apps. 
	
	[client loginWithProvider:@"microsoftaccount" 
		token:@{@"authenticationToken" : self.liveClient.session.authenticationToken}
		completion:^(MSUser *user, NSError *error) {
			self.loggedInUser = user;
			NSString *msg = [@"You are logged in as " stringByAppendingString:user.userId];
			UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Login" 
				message:msg 
				delegate:nil 
				cancelButtonTitle:@"OK" 
				otherButtonTitles: nil];
			[alert show];
	}];

This code assumes that you have previously created a **LiveConnectClient** instance named `liveClient` in the controller and the user is logged in.

###<a name="caching-tokens"></a>How to: Cache authentication tokens

To prevent users from having to authenticate every time they use run your application, you can cache the current user identity after they log in. You can then use this information to create the user directly and bypass the login process.  To do this you must store the user ID and authentication token locally. In the following example, the token is cached securely in the [KeyChain]:

	- (NSMutableDictionary *) createKeyChainQueryWithClient:(MSClient *)client andIsSearch:(bool)isSearch
	{
		NSMutableDictionary *query = [[NSMutableDictionary alloc] init];
		[query setObject:(__bridge id)kSecClassInternetPassword forKey:(__bridge id)(kSecClass)];
		[query setObject:client.applicationURL.absoluteString forKey:(__bridge id)(kSecAttrServer)];
    	
		if(isSearch) {
			// Use the proper search constants, return only the attributes of the first match.
			[query setObject:(__bridge id)kSecMatchLimitOne forKey:(__bridge id)kSecMatchLimit];
			[query setObject:(id)kCFBooleanTrue forKey:(__bridge id)kSecReturnAttributes];
			[query setObject:(id)kCFBooleanTrue forKey:(__bridge id)kSecReturnData];
		}
    
		return query;
	}

	- (IBAction)loginUser:(id)sender {
		NSMutableDictionary *query = [self createKeyChainQueryWithClient:self.todoService.client andIsSearch:YES];
		CFDictionaryRef cfresult;

		OSStatus status = SecItemCopyMatching((__bridge CFDictionaryRef)query, (CFTypeRef *)&cfresult);
		if (status == noErr) {
			NSDictionary * result = (__bridge_transfer NSDictionary*) cfresult;
			
			//create an MSUser object
			MSUser *user = [[MSUser alloc] initWithUserId:[result objectForKey:(__bridge id)(kSecAttrAccount)]];
			NSData *data = [result objectForKey:(__bridge id)(kSecValueData)];
			user.mobileServiceAuthenticationToken = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
			[self.todoService.client setCurrentUser:user];
			
		} else if (status == errSecItemNotFound) {
			//we need to log the user in
			[self.todoService.client loginWithProvider:@"MicrosoftAccount" controller:self animated:YES
				completion:^(MSUser *user, NSError *error) {
					NSString *msg = [@"You are logged in as " stringByAppendingString:user.userId];
					UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Login" 
											message:msg delegate:nil 
											cancelButtonTitle:@"OK" 
											otherButtonTitles: nil];
					[alert show];
                
					//save the user id and token to the KeyChain
					NSMutableDictionary *newItem = [self createKeyChainQueryWithClient:self.todoService.client 
															andIsSearch:NO];
					[newItem setObject:user.userId forKey:(__bridge id)kSecAttrAccount];
					[newItem setObject:[user.mobileServiceAuthenticationToken dataUsingEncoding:NSUTF8StringEncoding] 
                                                    forKey:(__bridge id)kSecValueData];
                 
					OSStatus status = SecItemAdd((__bridge CFDictionaryRef)newItem, NULL);
					if(status != errSecSuccess) {
						//handle error as needed
						NSAssert(NO, @"Error caching password.");
					}
			}];
		}	

<div class="dev-callout"><strong>Note</strong>
<p>Tokens are sensitive data and must be stored encrypted in case the device is lost or stolen.</p>
</div>

When using a cached token, a user will not have to login again until the token expires. When a user tries to login with an expired token, a 401 unauthorized response is returned. At this point, the user must log in again to obtain a new token, which can again be cached. You can use filters to avoid having to write code that handles expired tokens wherever your app calls the mobile service.  Filters allow you to intercept calls to and responses from your mobile service. The code in the filter tests the response for a 401, triggers the login process if the token is expired, and then retries the request that generated the 401. For details, see [Handling Expired Tokens].

<h2><a name="errors"></a><span class="short-header">Error handling</span>How to: Handle errors</h2>

When a call is made to the mobile service, the completion block contains an `NSError *error` parameter. When an error occurs, this parameter is returned a non-null value. In your code, you should check this parameter and handle the error as needed.

When an error has occurred, you can get more information by including the MSError.h file in the code. This file defines the following constants you can use to access additional data from `[error userInfo]`:

+ **MSErrorResponseKey**: the HTTP response data associated with the error
* **MSErrorRequestKey**: the HTTP request data associated with the error

In addition, a constant is defined for each error code. An explanation of these codes can be found in the MSError.h file.

For an example of performing validation and handling any, see [Validate and modify data in Mobile Services by using server scripts]. In this topic, server-side validation is implemented by using server scripts. When invalid data is submitted, and error response is returned and this response is handled by the client.

<!--
<h2><a name="#unit-testing"></a><span class="short-header">Designing tests</span>How to: Design unit tests</h2>

_(Optional) This section shows how to write unit test when using the client library (info from Yavor)._

<h2><a name="#customizing"></a><span class="short-header">Customizing the client</span>How to: Customize the client</h2>

_(Optional) This section shows how to send customize client behaviors._

###<a name="custom-headers"></a>How to: Customize request headers

_(Optional) This section shows how to send custom request headers._

For more information see, New topic about processing headers in the server-side.

###<a name="custom-serialization"></a>How to: Customize serialization

_(Optional) This section shows how to use attributes to customize how data types are serialized._

For more information see, New topic about processing headers in the server-side.

## <a name="next-steps"></a>Next steps
-->

<!-- Anchors. -->

[What is Mobile Services]: #what-is
[Concepts]: #concepts
[Setup and Prerequisites]: #Setup
[How to: Create the Mobile Services client]: #create-client
[How to: Create a table reference]: #table-reference
[How to: Query data from a mobile service]: #querying
[Filter returned data]: #filtering
[Sort returned data]: #sorting
[Return data in pages]: #paging
[Select specific columns]: #selecting
[How to: Bind data to the user interface]: #binding
[How to: Insert data into a mobile service]: #inserting
[How to: Modify data in a mobile service]: #modifying
[How to: Authenticate users]: #authentication
[Cache authentication tokens]: #caching-tokens
[How to: Upload images and large files]: #blobs
[How to: Handle errors]: #errors
[How to: Design unit tests]: #unit-testing 
[How to: Customize the client]: #customizing
[Customize request headers]: #custom-headers
[Customize data type serialization]: #custom-serialization
[Next Steps]: #next-steps
[How to: Use MSQuery]: #query-object

<!-- Images. -->

<!-- URLs. -->
[Get started with Mobile Services]: ../tutorials/mobile-services-get-started-iOS.md
[Validate and modify data in Mobile Services by using server scripts]: ../tutorials/mobile-services-validate-and-modify-data-ios.md
[Mobile Services SDK]: https://go.microsoft.com/fwLink/p/?LinkID=266533
[Get started with authentication]: ../tutorials/mobile-services-get-started-with-users-iOS.md
[iOS SDK]: https://developer.apple.com/xcode
[KeyChain]: http://go.microsoft.com/fwlink/p/?LinkId=301953
[Handling Expired Tokens]: http://go.microsoft.com/fwlink/p/?LinkId=301955
[Live Connect SDK]: http://go.microsoft.com/fwlink/p/?LinkId=301960
[Permissions]: http://msdn.microsoft.com/en-us/library/windowsazure/jj193161.aspx
[Use scripts to authorize users]: ../tutorials/mobile-services-authorize-users-ios.md
[Dynamic schema]: http://go.microsoft.com/fwlink/p/?LinkId=296271
[How to: access custom parameters]: ../howto/mobile-services-work-with-server-scripts.md#access-headers
[Create a table]: http://msdn.microsoft.com/en-us/library/windowsazure/jj193162.aspx
[NSDictionary object]: http://go.microsoft.com/fwlink/p/?LinkId=301965