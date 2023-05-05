# Writeup for d4icu

Among the various session persistence solutions available for Tomcat, one approach is to store session data in Redis.

This challenge uses the `tomcat-cluster-redis-session-manager` library, available at https://github.com/ran-jit/tomcat-cluster-redis-session-manager.

If we read the source code, we can find that session data is serialized using the serialization functionality provided by the JDK before being written to Redis. When reading the data, it undergoes deserialization.

In Redis, the stored format is `session_id: serialized_binary_data.`
The value of the session ID is the value of the `JSESSIONID` cookie.
If an attacker can write arbitrary data to Redis, they can carry out a deserialization attack.

To make the attack feasible, we added `CommonsCollections 3.1` as a dependency.

Coincidentally, the cache program provides caching functionality that can cache the content of an HTTP response to Redis.

Later, there will be a headless browser that can access `/demo/index.jsp` in Tomcat and capture a screenshot to return to the user.
This is a Node.js application. If you have good reading habits, you may have noticed in `package.json` that the version of `puppeteer` is very old. Each version of `puppeteer` is only compatible with a specific version of Chromium. The current version of `puppeteer` used is 6.0.0, which corresponds to Chromium version 89.
This version of Chromium has a remote code execution vulnerability (CVE-2021-21220).

Specific solution steps:

1. Use cache service to write maliciously crafted binary data to Redis.
2. Modify the cookie to trigger deserialization and gain RCE privilege on the Tomcat container.
3. Modify Index.jsp to implement a watering hole attack.
4. Make Chromium access index.jsp, get RCE, and obtain the flag.

In addition, the load balancing is configured in the question, with a total of three Tomcat containers providing HTTP services. To trigger the RCE of Chromium stably, the index.jsp of all three Tomcat containers needs to be rewritten.

Here are some references:

- https://github.com/ran-jit/tomcat-cluster-redis-session-manager
- https://commons.apache.org/proper/commons-collections/
- https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-21220
- https://cjovi.icu/CVE/1586.html


If you have any questions or suggestions, please feel free to contact me(https://t.me/ElisaOmega).(https://t.me/ElisaOmega).
