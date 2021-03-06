# TwitterStream

![go twitter](./go-twitter.png)

[![Go Report Card](https://goreportcard.com/badge/github.com/fallenstedt/twitter-stream)](https://goreportcard.com/report/github.com/fallenstedt/twitter-stream)
[![Go Reference](https://pkg.go.dev/badge/github.com/fallenstedt/twitter-stream.svg)](https://pkg.go.dev/github.com/fallenstedt/twitter-stream)

TwitStream is a Go library for streaming tweets with [Twitter's v2 Filtered Streaming API](https://developer.twitter.com/en/docs/twitter-api/tweets/filtered-stream/introduction).

![example of twit stream](./example.gif)

This project is not production ready. There are several things I need to do: 
- [ ] This package streams strings. I need to convert json into go structs with [these possible response fields](https://developer.twitter.com/en/docs/twitter-api/tweets/filtered-stream/api-reference/get-tweets-search-stream)  


## Installation

`go get github.com/fallenstedt/twitter-stream`


## Examples

#### Starting a stream

Once you obtain an Access Token, you can create a TwitterStream instance with `NewTwitterStream(accesToken)`
Then, you can invoke `StartStream` to begin the streaming process. 

To read messages from your stream, start a loop with `GetMessages`. The messages that is returned could be
a tweet, or an error. 

The possible errors that can be returned are 
* `io.EOF`: An error that you have reached the end of the stream. 
* `non io.EOF errors`: This could be errors that are returned from Twitter during your stream

[You can learn more about processing data by reading Twitter's documentation here](https://dev.twitter.com/streaming/overview/processing)

```go
// Starting a stream assuming you already have
// stream rules set in place
func startStreaming() {
	// Obtain an AccessToken
	// You can use the token generator and provide your api key and secret
	// or provide an access token you already have
	token, err := twitterstream.NewTokenGenerator().SetApiKeyAndSecret(
		"your_twitter_api_key",
		"your_twitter_api_secret",
	).RequestBearerToken()

	if err != nil {
		panic("No token found!")
	}

	// With an access token, you can create a new twitterstream and start streaming
	api := twitterstream.NewTwitterStream(token.AccessToken)
	err := api.Stream.StartStream()
	if err != nil {
		panic(err)
	}

	// If you do not put this in a go routine, you will stream forever
	go func() {
		// Range over the messages channel to get a message, or an error.
		for message := range *api.Stream.GetMessages() {
			fmt.Println(message)
		}
	}()

	// After 30 seconds, stop the stream
	time.Sleep(time.Second * 30)
	api.Stream.StopStream()
}
```

#### Creating, Deleting, and Getting Rules

```go


func addRules() {
	// Obtain an AccessToken
	// You can use the token generator and provide your api key and secret
	// or provide an access token you already have
	token, err := twitterstream.NewTokenGenerator().SetApiKeyAndSecret(
		"your_twitter_api_key",
		"your_twitter_api_secret",
	).RequestBearerToken()

	if err != nil {
		panic("No token found!")
	}

	// With an access token, you can create a new twitterstream and start adding rules
	api := twitterstream.NewTwitterStream(token.AccessToken)


	// You can add rules by passing in stringified JSON with the rules you want to add
	// You can learn more about building rules here: https://developer.twitter.com/en/docs/twitter-api/tweets/filtered-stream/integrate/build-a-rule
	// Or here: https://developer.twitter.com/en/docs/twitter-api/tweets/filtered-stream/api-reference/post-tweets-search-stream-rules
	// The response are the rules you created
	// The 2nd argument will perform a dry run if set to true.
	res, err := api.Rules.AddRules(`{
		"add": [
				{"value": "cat has:images", "tag": "cat tweets with images"}
			]
		}`, false)

	if err != nil {
		panic(err)
	}

	fmt.Println(res.Data, res.Meta)
}

func deleteRules() {
	// Obtain an AccessToken
	// You can use the token generator and provide your api key and secret
	// or provide an access token you already have
	token, err := twitterstream.NewTokenGenerator().SetApiKeyAndSecret(
		"your_twitter_api_key",
		"your_twitter_api_secret",
	).RequestBearerToken()

	if err != nil {
		panic("No token found!")
	}

	// With an access token, you can create a new twitterstream and start deleting rules
	api := twitterstream.NewTwitterStream(token.AccessToken)

	// You can delete rules by passing in stringified JSON with the rules you want to delete
	// Learn more about deleting rules here: https://developer.twitter.com/en/docs/twitter-api/tweets/filtered-stream/api-reference/post-tweets-search-stream-rules
	// The ids are the rule's ids you want to delete. You can find out how to get your ids in the below example
	// The response are the rules you have.
	// The 2nd argument will perform a dry run if set to true.
	res, err := api.Rules.AddRules(`{
		"delete": {
				"ids": ["1340894899986579457"]
			}
		}`, false)

	if err != nil {
		panic(err)
	}

	fmt.Println(res.Data, res.Meta)
}

func getRules() {
	// Obtain an AccessToken
	// You can use the token generator and provide your api key and secret
	// or provide an access token you already have
	token, err := twitterstream.NewTokenGenerator().SetApiKeyAndSecret(
		"your_twitter_api_key",
		"your_twitter_api_secret",
	).RequestBearerToken()

	if err != nil {
		panic("No token found!")
	}

	// With an access token, you can create a new twitterstream and start getting your rules
	api := twitterstream.NewTwitterStream(token.AccessToken)

	// You can get your rules by invoking GetRules
	// Learn more about getting rules here: https://developer.twitter.com/en/docs/twitter-api/tweets/filtered-stream/api-reference/get-tweets-search-stream-rules
	res, err := api.Rules.GetRules()

	if err != nil {
		panic(err)
	}

	fmt.Println(res.Data, res.Meta)
}

```



## Contributing

Pull requests are always welcome. Please accompany a pull request with tests. 

