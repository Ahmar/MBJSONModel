#MBJSONModel

Quick and lightweight JSON → NSObject translation.

Example
-------
```
@interface User : MBJSONModel

@property (nonatomic, copy) NSString *name;
@property (nonatomic) NSDate *birthDate;
@property (nonatomic) NSArray *tweets;

@end

@implementation User

+ (NSDictionary *)JSONKeyTranslationDictionary
{
    return @{
				@"user_name" : @"name",
				@"birth_date" : @{@"isDate" : @YES, @"format" : @"yyyy-MM-dd", @"property" : @"birthDate"},
				@"data.user.tweets" : @{@"class" : NSStringFromClass([Tweet class]), @"relationship" : @"tweets", @"isArray" : @YES},
              }
}

@end
```

Then, if you have a JSON dictionary that looks like this:
```
{
	"user_name" : "Jon Snow",
	"birth_date" : "1970/03/29",
	"data" : {
		"user" : {
			"tweets" : [
				{
					"text" : "hello world"
				}
			]
		}
	}
}
```

You can easily do:
```
User *userObj = [User modelFromJSONDictionary:jsonDict];
```


Advanced Options
-------
You can override a transformer method to manually transform a JSON object to native. Say you have the following JSON dictionary:
```
{
	"text" : "hello\nworld"
}
```

and the following translation dictionary

```
+ (NSDictionary *)JSONKeyTranslationDictionary
{
    return @{
				@"custom_text" : @"text",
            }
}
```

and wanted to change the text to "hello world" before translating to native, you can implement

```
- (MBValueTransformer *)textJSONValueTransformer
{
	return [MBValueTransformer transformerWithBlock:^id(NSString *incomingString) {
       return [incomingString stringByReplacingOccurrencesOfString:@"/n" withString:@" "];
    }];
}
```

This method will automatically be called by MBJSONModel during the translation process.

####NSCopying
MBJSONModels conform to NSCopying, which means you can make a copy of any model object:

```
User *user = [User modelFromJSONDictionary:@{@"user_name" : @"Jon Snow"}];
User *userCopy = [user copy];
assert([user.name isEqual:userCopy.name]); // Passes
```

####NSObject → NSDictionary

There are two dictionary instance methods:

```
/**
 Returns a dictionary where keys are named exactly as the @property is named
 */
- (NSDictionary *)dictionaryFromObjectProperties;
/**
 Returns a dictionary where keys are converted to JSONKeys using -JSONKeyForPropertyName:
 */
- (NSDictionary *)JSONDictionaryRepresentation;
```

```
User *user = [User modelFromJSONDictionary:@{@"user_name" : @"Jon Snow"}];
```

```
NSLog([user dictionaryFromObjectProperties]);
```

Output:
> {"name" : "Jon Snow"}

```
NSLog([user JSONDictionaryRepresentation]);
```

Output:
> {"user_name" : "Jon Snow"}

Contact
-------

**Mo Bitar** | [@bitario](https://twitter.com/bitario)

License
-------
MBJSONModel is available under the MIT license.