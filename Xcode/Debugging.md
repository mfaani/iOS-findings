# OSLog, Debugging Profiles, APNs
So we reached out to Apple about Push notifications where the images get downloaded and we're able to store the image successfully. However the image still doesn't show up sometims. 
They recommendation was to use `OSLog`. Add your own logs around everything and use that as a _marker_ next to other system logs. 

Focus on APSd, All board like processes. But mainly just filter by your your bundle Id. 

Also might want to turn on extra logging by adding certain debugging profiles. We were recommended to add the following debug profiles
- Messages
- APNs
- Notifications

Each profile will expire after 2-3 days to avoid extra logging to protect space and privacy.

https://developer.apple.com/bug-reporting/profiles-and-logs/
