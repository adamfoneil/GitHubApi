This started as a code blogging solution, but then I found [Micro.blog](https://micro.blog/) and I got *all the way over* building my own blogging platform. The part of this original plan worth keeping, however IMO, is a C# [GitHub API client](https://github.com/adamfoneil/GitHubApi/blob/master/GitHubApi.Library/GitHubApiClient.cs) implemented with [Refit](https://github.com/reactiveui/refit). My Refit interface is [here](https://github.com/adamfoneil/GitHubApi/blob/master/GitHubApi.Library/Interfaces/IGitHubApi.cs). This is certainly a work in progress because I've implemented only a few methods, and the whole [GitHub API](https://docs.github.com/en/free-pro-team@latest/rest) is rather large. I'm not sure how much more I'll do in the short term. Yes I'm aware of [Octokit](https://www.nuget.org/packages/Octokit), GitHub's own API client. If you needed something full-featured, you'd use that.

I wanted more practice using Refit because there are some slightly tricky things about getting it working.

- I'm using access token authentication. To get Refit to use a token for authentication, I added an [AuthorizationHeaderValueGetter](https://github.com/adamfoneil/GitHubApi/blob/master/GitHubApi.Library/GitHubApiClient.cs#L44) delegate, which simply returns the token that was passed through the client object constructor. The less obvious part is that you need to add a `[Headers]` attribute on your Refit interface that is formed a certain way. In particular, you need [Authorization: token](https://github.com/adamfoneil/GitHubApi/blob/master/GitHubApi.Library/Interfaces/IGitHubApi.cs#L8) as one of the headers. It appears that the token you provide gets concatenated to that header. There are a couple other headers GitHub looks for: `User-Agent` (required) and `Accept` (optional).

- A frustration I ran into was that I could get Postman to work with sample API calls, but then I wasn't sure how to form the C#-Refit equivalent exactly. I had to just try seveeral things regarding that `[Header]` attribute. I eventually got it working.

- Note that my Refit [interface](https://github.com/adamfoneil/GitHubApi/blob/master/GitHubApi.Library/Interfaces/IGitHubApi.cs#L9) itself is `internal`. This is because the raw API endpoints aren't the easiest things to use from a C# perspective. There are a couple reasons for this. For one, they use string arguments where `enum`s would work better. My [public client object](https://github.com/adamfoneil/GitHubApi/blob/master/GitHubApi.Library/GitHubApiClient.cs) uses private enum converter methods [like this](https://github.com/adamfoneil/GitHubApi/blob/master/GitHubApi.Library/GitHubApiClient.cs#L85). Second, to get all items in a paginated list, you need some kind of wrapper method that [enumerates all pages](https://github.com/adamfoneil/GitHubApi/blob/master/GitHubApi.Library/GitHubApiClient.cs#L62).

- For testing purposes, I'm using my own access token, which I need to keep out of source control. So, if you want to clone and run this repo, you'll need to create your own access token and set it up in the repo just so. Create a json file called **github.json** inside a Config folder. Your config file should look like this:

![img](https://adamosoftware.blob.core.windows.net/images/github-api-config.png)

The config content is accessed through this [Config](https://github.com/adamfoneil/GitHubApi/blob/master/GitHubApi.Tests/GitHubIntegration.cs#L47) property, and then I have a [factory method](https://github.com/adamfoneil/GitHubApi/blob/master/GitHubApi.Tests/GitHubIntegration.cs#L45) for my client object that uses this configuration.
