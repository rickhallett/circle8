
                           Daisys API documentation

   This is the documentation for the public wrapper library for the Daisys
   API. The main product is “Speak”, which provides text-to-speech (TTS)
   services.

   Have your product talking in seconds!

   Building with an LLM? Give it this single-file text version of these docs.

   Example

 from daisys import DaisysAPI
 from daisys.v1.speak import SimpleProsody

 with DaisysAPI('speak', email='user@example.com', password='pw') as speak:
     voice = await speak.get_voices()[-1]
     print(f"{voice.name} speaking!")
     take = speak.generate_take(voice_id=voice.voice_id, text="Hello there, I am Daisys!",
                                prosody=SimpleProsody(pace=-3, pitch=2, expression=10))
     audio_wav = speak.get_take_audio(take.take_id, filename='hello_daisys.wav')

Getting started

   Once you confirm your email address, you will be provided access to the
   Daisys API via the email account you registered. You have also already
   provided a password.

   At this point the API can be used. The steps are:

    1. Authentication: provide your email and password to get an access
       token.

    2. List the models that are available to you.

    3. Create a voice for a model.

    4. Create a “take” with that voice. (Request some speech.)

    5. Download the audio for that take.

   All these steps are taken care of by the Python client library. Therefore,
   first we show briefly how to use the library, and secondly we show how to
   accomplish the above steps manually from the command line using curl. This
   can be useful if you want to build your own client in your preferred
   language.

   Depending on your needs, you may prefer not to provide the email and
   password every time the API is used. It is also possible refresh the
   access token using the provided refresh token, thereby continuing a
   previous session without logging out.

   For more on using the Python client library, see Daisys API examples.

  Getting started with the Python client library

   For more on using the Python client library, see Daisys API examples.

   In the following, the default “synchronous” client will be demonstrated.
   Some users will prefer to use asyncio, and in the following examples, with
   DaisysAPI() can be replaced with async with DaisysAPI, which returns an
   asynchronous client library that can be used with the await keyword.

    Installing the library

   The library is available on pypi.org and can be installed via pip. The
   Daisys API requires Python version 3.10 or greater. First create a Python
   venv, activate it, install the library, and then download and run the
   examples:

   Installing the library

 $ # setup a virtual environment
 $ mkdir daisys_project
 $ cd daisys_project
 $ python3 -m venv venv
 $ . venv/bin/activate
 $
 $ # install the library
 $ python3 -m pip daisys
 $
 $ # or if Python websocket support is needed
 $ python3 -m pip daisys[ws]

   Of course pip is only one option, you can use any Python project
   management software such as uv, Poetry, etc.

    Running an example

   Within the Python virtual environment, the hello_daisys.py example can be
   run. The examples are programmed to take your email and password in the
   environment variables as shown:

   Running an example

 $ curl -O https://raw.githubusercontent.com/daisys-ai/daisys-api-python/main/examples/hello_daisys.py
 $ export DAISYS_EMAIL=user@example.com
 $ export DAISYS_PASSWORD=example_password123
 $ python3 hello_daisys.py

    Getting a client

   Getting a client by context manager

 from daisys import DaisysAPI
 with DaisysAPI('speak', email='user@example.com', password='pw') as speak:
     ...

 # or for asyncio support:
 async with DaisysAPI('speak', email='user@example.com', password='pw') as speak:
     ...

   As mentioned, an asyncio-enabled client can be instantiated by using async
   with in the line above. Additionally, the context manager interface (with)
   is optional; it is also possible to create a client by normal function
   call:

   Getting a client by function

 from daisys import DaisysAPI
 speak = DaisysAPI('speak', email=EMAIL, password=PASSWORD).get_client()
 # or..
 speak = DaisysAPI('speak', email=EMAIL, password=PASSWORD).get_async_client()

   The main difference is that when an email and password are used, the
   context manager approach will automatically log out when the program exits
   the context, whereas when the client is retrieved by get_client or
   get_async_client, then .logout() function should be called. Logging out
   invalidates the refresh token so that no further sessions can be renewed
   without logging in again. Auto-logout will not occur when an access token
   is provided.

   The rest of this documentation will assume the normal, synchronous client.
   In all cases, functions should be called with await when used with the
   asyncio client.

    Listing the models

   Using the client library, it is easy to log into the API and start
   requesting text to speech services. The following Python code can be used
   to list the available models:

   Listing the models

 from daisys import DaisysAPI
 with DaisysAPI('speak', email='user@example.com', password='pw') as speak:
     print('Found models:')
     for model in speak.get_models():
         print(model)

    Listing the voices

   You can use a model by using a voice associated with that model. Voices
   are identified by a voice_id field.

   Listing the voices

 from daisys import DaisysAPI
 with DaisysAPI('speak', email='user@example.com', password='pw') as speak:
     print('Found voices:')
     for voice in speak.get_voices():
         print(f'{voice.name}, a {voice.gender} voice of {voice.model} with id {voice.voice_id}.')

    Generating a voice

   If you do not yet have any voices, you should generate one. Voices can be
   requested for a given gender and with default prosody information. Voices
   must be given names.

   For instance, the following block of code creates an expressive female
   voice for the shakespeare model:

   Generating a voice

 from daisys import DaisysAPI, VoiceGender
 from pprint import pprint
 with DaisysAPI('speak', email='user@example.com', password='pw') as speak:
     print('Creating a voice:')
     voice = speak.generate_voice(name="Deirdre", gender=VoiceGender.FEMALE, model="shakespeare")
     pprint(voice.model_dump())

   Note that voice generation can take a few seconds! In this example, the
   speak.generate_voice command waits for the operation to finish, and
   therefore we can print the result immediately.

   It is also possible to adopt a more asynchronous style by providing
   wait=False to speak.generate_voice(). Alternatively, as mentioned above
   you can use the asyncio client to allow the await speak.generate_voice()
   syntax.

   The above code gives the following details:

   Generating a voice: output

 Creating a voice:
 {'default_style': [],
  'default_prosody': None,
  'done_webhook': None,
  'example_take': None,
  'example_take_id': 't01hasgezqkx4vth62xckymk3x3',
  'gender': <VoiceGender.FEMALE: 'female'>,
  'model': 'shakespeare',
  'name': 'Deirdre',
  'status': <Status.READY: 'ready'>,
  'timestamp_ms': 1695218371261,
  'voice_id': 'v01hasgezqjcsnc91zdfzpx0apj'}

   We can see that the voice has a female gender, and has an example take
   associated with it. This take_id can already be used to hear the voice.

    Generating a take

   Now that you have a voice, text to speech can be requested by the
   speak.take_generate() command:

   Generating a take

 from daisys import DaisysAPI
 from pprint import pprint
 with DaisysAPI('speak', email='user@example.com', password='pw') as speak:
     print('Creating a take:')
     take = speak.generate_take(voice_id='v01hasgezqjcsnc91zdfzpx0apj',
                                text="Hello, Daisys! It's a beautiful day.")
     pprint(take.model_dump())

   Giving,

   Generating a take: output

 Creating a take:
 {'done_webhook': None,
  'info': {'audio_rate': 44100,
           'duration': 152576,
           'normalized_text': ['Hello, Daisys!', "It's a beautiful day."]},
  'override_language': None,
  'prosody': None,
  'status': <Status.READY: 'ready'>,
  'status_webhook': None,
  'style': None,
  'take_id': 't01hasgn2dnyg6jqrcym9cgxv75',
  'text': "Hello, Daisys! It's a beautiful day.",
  'timestamp_ms': 1695220926901,
  'voice_id': 'v01hasgezqjcsnc91zdfzpx0apj'}

   Note that the status is “ready”, meaning that audio can now be retrieved.
   As with voice generation, an asynchronous approach is also available for
   generate_take.

    Retrieving a take’s audio

   The take is ready, now we can hear the result! Audio for a take can be
   retrieved as follows:

   Retrieving audio (1)

 from daisys import DaisysAPI
 with DaisysAPI('speak', email='user@example.com', password='pw') as speak:
     print("Getting a take's audio.")
     audio_wav = speak.get_take_audio(take_id='t01hasghx0zgdc29gpzexw5r8wc', file='beautiful_day.wav')
     print('Length in bytes:', len(audio_wav))

   In the above code, we retrive a .wav file, which is (optiionally) written
   to a file in addition to being returned. This can be decoded for example
   using scipy’s io.wavfile module:

   Retrieving audio (2)

     from scipy.io import wavfile
     from io import BytesIO
     print(wavfile.read(BytesIO(audio_wav)))

     # Note: Since decoding the audio is outside the scope of the client library,
     # `scipy` is not a dependency and will not be automatically installed by `pip`.

   which, along with the previous code block, prints:

   Retrieving audio: output

 Getting a take's audio.
 Length in bytes: 292908
 (44100, array([-111,  -46, -104, ..., -128,  -95,   -9], dtype=int16))

   The resulting file beautiful_day.wav can be played using command line
   programs like aplay on Linux, or any audio player such as the excellent
   VLC. You can integrate the results into your creative projects!

   It is also possible to retrieve the audio in other formats: mp3, flac, and
   m4a by providing the format parameter.

    Streaming audio

   The Daisys API supports two methods of streaming audio:

     * HTTP

     * Websocket

      HTTP

   The HTTP method downloads the audio file in chunks using a streaming
   response, and can be convenient if a simple iterator interface is desired.
   When making the take request, set wait to False, and call
   stream_take_audio() (async stream_take_audio()). Alternatively a signed
   URL can be retrieved using get_take_audio_url() (async
   get_take_audio_url()), useful for passing to an audio playing running on a
   frontend browser.

   Streaming audio, HTTP method

 from daisys import DaisysAPI
 with DaisysAPI('speak', email='user@example.com', password='pw') as speak:
     print("Streaming a take's audio.")
     with speak.stream_take_audio(take_id='t01hasghx0zgdc29gpzexw5r8wc') as stream:
         for chunk in stream:
             print('Length in bytes:', len(chunk))

   When using the HTTP method via endpoints outside of the Python library,
   please be aware of the use of 307 redirects and headers, outlined in
   Retrieving audio.

      Websocket

   See Daisys API websocket examples.

   For lowest latency usage, it is additionally possible to use a websocket
   to create a connection directly to the worker node used for synthesizing
   audio. Requests are submitted to the worker and the same node streams back
   the audio as it is generated over the already-established connection.

   Streaming audio, websocket method

 from daisys import DaisysAPI
 with DaisysAPI('speak', email='user@example.com', password='pw') as speak:
     print("Streaming a take's audio.")
     with speak.websocket(voice_id='v01hasgezqjcsnc91zdfzpx0apj') as ws:
         request_id = ws.generate_take(voice_id='v01hasgezqjcsnc91zdfzpx0apj',
                                       text="Hello, Daisys! It's a beautiful day.",
                                       audio_callback=my_audio_cb,
                                       status_callback=my_status_cb)

   The specified callbacks will be called whenever the requested take’s
   status changes or audio data is generated. See Example: Websocket example,
   synchronous client for complete information on the signatures of these two
   callbacks and examples showing how they can be used to receive audio in
   chunks as it is generated.

   In addition to the callback interface, iter_request() (async
   iter_request()) is provided to allow an iterator-based for-loop (or async
   for-loop) over incoming audio chunks, simplifying usage.

   Finally, in applications where the backend should perform REST API calls
   but the front-end should stream audio, websocket_url() can be used to
   retrieve a URL that the front-end should connect a websocket to. Example:
   Websocket example, web client is provided to show how to manage the
   websocket connection using JavaScript.

    Authentication with access tokens

   All the above examples authenticate with the API using email and password.
   In some scenarios users will prefer to authenticate using only the access
   token. An access and refresh token can be retrieved once and used until it
   is manually revoked.

   By default, when the client library is used with email and password, the
   refresh token is automatically revoked when the client context is exited.
   When an access token is provided to the client context, this automatic
   revocation is skipped, so that the token can be refreshed on next usage.
   This can be controlled by setting speak.auto_logout to True or False.

   To retrieve an access and refresh token for future use, the following
   program can thus be used:

   Retrieving an access and refresh token

 from daisys import DaisysAPI
 with DaisysAPI('speak', email='user@example.com', password='pw') as speak:
     speak.auto_logout = False
     speak.login()
     access_token, refresh_token = speak.access_token, speak.refresh_token

   These tokens can now be stored, and provided to the client as follows:

   Retrieving an access and refresh token

 from daisys import DaisysAPI

 def store_tokens(speak, access_token: str, refresh_token: str):
     """Store the current Daisys access and refresh tokens."""
     with open('daisys_tokens.json','w') as token_file:
         json.dump([access_token, refresh_token], token_file)

 access_token, refresh_token = json.load(open('daisys_tokens.json'))
 with DaisysAPI('speak', access_token=access_token, refresh_token=refresh_token) as speak:
     speak.token_callback = store_tokens
     ...

   The library does not implement a storage and retrieval mechanism for these
   tokens, as it is presumed that users will have their own files or
   databases for this purpose.

   Importantly, when an access token expires, a new one will be automatically
   retrieved by the library. Therefore, it is useful to store
   speak.access_token and speak.refresh_token whenever it changes. The
   token_callback is provided for this purpose. It is optional, but
   recommended if not using a permatoken and one wishes to avoid transmitting
   passwords.

  Getting started with the command line

   The Daisys API can be used from the command line using curl and jq. Most
   application writers will want to use this guide to see how to make HTTP
   calls to the API for developing their own client libraries in their
   favorite language.

    Running the curl example

   The Python client library source code bundles an example of how to use the
   API this way. Instructions to run that example are provided on the linked
   page.

   The rest of this document shall describe how to use the API one step at a
   time in a shell, rather than in a shell script. In the examples, the
   result of curl is piped to jq . for formatting purposes.

    Authenticating

   To access the Daisys Speak API, you must attach an access token to any
   HTTP calls, with the exception of the /version endpoint.

   To get such an access key, it can be requested by providing an email and
   password as follows:

   Authenticating: Getting an access token

 TOKENS=$(curl -s -X POST -H 'content-type: application/json' \
          -d '{"email": "user@example.com", "password": "my_password123"}' \
          https://api.daisys.ai/auth/login)
 export ACCESS_TOKEN=$(echo $TOKENS | jq -r .access_token)
 export REFRESH_TOKEN=$(echo $TOKENS | jq -r .refresh_token)

   You can keep using this access token for a limited time. It can be used by
   adding it into the string Bearer $ACCESS_TOKEN for the value of the
   Authorization header.

   If you receive a 401 response from any API request, the access token needs
   to be refreshed by issuing:

   Authenticating: Refreshing the access token

 $ TOKENS=$(curl -s -X POST -H 'content-type: application/json' \
            -H "Authorization: Bearer $ACCESS_TOKEN" \
            -d '{"refresh_token": "'$REFRESH_TOKEN'"}' \
            https://api.daisys.ai/auth/refresh)
 $ export ACCESS_TOKEN=$(echo $TOKENS | jq -r .access_token)
 $ export REFRESH_TOKEN=$(echo $TOKENS | jq -r .refresh_token)

    Listing the models

   Models can be listed by accessing the /models endpoint. More information
   on the options are found in Model-related Endpoints.

   Listing the models

 $ curl -s -X GET -H "Authorization: Bearer $ACCESS_TOKEN" https://api.daisys.ai/v1/speak/models | jq .
 [
   {
     "name": "shakespeare",
     "displayname": "Shakespeare",
     "flags": [],
     "languages": [
       "en-GB"
     ],
     "genders": [
       "female",
       "male"
     ],
     "styles": [
       [
         "base",
         "character",
         "narrator"
       ]
     ],
     "prosody_types": [
       "simple",
       "affect"
     ]
   }
 ]

    Listing the voices

   Voices can be listed by accessing the /voices endpoint. More information
   on the options are found in Voice-related Endpoints.

   Listing the voices

 $ curl -s -X GET -H "Authorization: Bearer $ACCESS_TOKEN" https://api.daisys.ai/v1/speak/voices | jq .
 [
   {
     "name": "Deirdre",
     "model": "shakespeare",
     "gender": "female",
     "default_style": [],
     "default_prosody": null,
     "example_take": null,
     "status_webhook": null,
     "done_webhook": null,
     "voice_id": "v01hasgezqjcsnc91zdfzpx0apj",
     "status": "ready",
     "timestamp_ms": 1695220727538,
     "example_take_id": "t01hasgezqkx4vth62xckymk3x3"
   }
 ]

    Generating a voice

   If you do not yet have any voices, you should generate one using the
   /voices/generate endpoint. Voices can be requested for a given gender and
   with default prosody information. Voices must be given names. More
   information on the options are found in Voice-related Endpoints.

   For instance, the following command creates an expressive female voice for
   the shakespeare model:

   Generating a voice

 $ curl -s -X POST -H 'content-type: application/json' \
 -H "Authorization: Bearer $ACCESS_TOKEN" \
 -d '{"name": "Ignacio", "gender": "male", "model": "shakespeare"}' \
 https://api.daisys.ai/v1/speak/voices/generate | jq .
 {
   "name": "Ignacio",
   "model": "shakespeare-pause_symbol-18-4-23",
   "gender": "male",
   "default_style": null,
   "default_prosody": null,
   "example_take": null,
   "done_webhook": null,
   "voice_id": "v01haxx5cggwz215gzv0hjbra9m",
   "status": "waiting",
   "timestamp_ms": 1695368262160,
   "example_take_id": "t01haxx5cgg3n8f2qzc8zkbn97y"
 }

   Note that voice generation can take a few seconds! In this example, the
   “status” is “waiting” and not yet “ready”, therefore we should check in on
   it again after a second or two. For this, we need to use the voice_id
   provided in the response:

   Checking the voice status

 $ curl -s -X GET -H 'content-type: application/json' \
 -H "Authorization: Bearer $ACCESS_TOKEN" \
 https://api.daisys.ai/v1/speak/voices/v01haxx5cggwz215gzv0hjbra9m | jq .
 {
   "name": "Ignacio",
   "model": "shakespeare-pause_symbol-18-4-23",
   "gender": "male",
   "default_style": null,
   "default_prosody": null,
   "example_take": null,
   "done_webhook": null,
   "voice_id": "v01haxx5cggwz215gzv0hjbra9m",
   "status": "ready",
   "timestamp_ms": 1695368262160,
   "example_take_id": "t01haxx5cgg3n8f2qzc8zkbn97y"
 }

   The voice is now “ready”! We can now get its example audio using the
   example_take_id field, see Retrieving a take’s audio below.

   Note: as seen in the response structure, a webhook can also be provided to
   get a notification when the result is ready. This webhook is called as a
   POST request with the same response structure as seen here, provided in
   the request body.

    Generating a take

   Now that you have a voice, text to speech can be requested by the
   /takes/generate endpoint. Here we generate one with default prosody for
   the voice, which we also left as default (neutral) when generating the
   voice above. More information on the options are found in Take-related
   Endpoints.

   Generating a take

 $ curl -s -X POST -H 'content-type: application/json' \
 -H "Authorization: Bearer $ACCESS_TOKEN" \
 -d '{"text": "Hello, Daisys! It'\''s a beautiful day.", "voice_id": "v01hasgezqjcsnc91zdfzpx0apj"}' \
 https://api.daisys.ai/v1/speak/takes/generate
 {
   "text": "Hello, Daisys! It's a beautiful day.",
   "override_language": null,
   "style": null,
   "prosody": null,
   "status_webhook": null,
   "done_webhook": null,
   "voice_id": "v01hasgezqjcsnc91zdfzpx0apj",
   "take_id": "t01haybgb16dn9dk0p5je47qz74",
   "status": "waiting",
   "timestamp_ms": 1695383301158,
   "info": null
 }

   Similar to with voice generation, take generation takes a couple of
   seconds, and the status can be retrieved by using the take_id:

   Generating a take: checking status

 $ curl -s -X GET -H "Authorization: Bearer $ACCESS_TOKEN" \
 https://api.daisys.ai/v1/speak/takes/t01haybgb16dn9dk0p5je47qz74 | jq .
 {
   "text": "Hello, Daisys! It's a beautiful day.",
   "override_language": null,
   "style": null,
   "prosody": null,
   "status_webhook": null,
   "done_webhook": null,
   "voice_id": "v01hasgezqjcsnc91zdfzpx0apj",
   "take_id": "t01haybgb16dn9dk0p5je47qz74",
   "status": "ready",
   "timestamp_ms": 1695383301158,
   "info": {
     "duration": 150528,
     "audio_rate": 44100,
     "normalized_text": [
       "Hello, Daisys!",
       "It's a beautiful day."
     ]
   }
 }

   Similar to voice generation, it is possible to use a webhook for the
   “done” notification. For longer texts, it is also possible to request a
   “status” webhook which may be called several times whenever the progress
   for a take changes.

   Here, we see the status is “ready”, meaning that audio can now be
   retrieved.

    Retrieving a take’s audio

   The take is ready, now we can hear the result! Audio for a take can be
   retrieved as follows:

   Retrieving audio

 $ curl -s -L -X GET -H "Authorization: Bearer $ACCESS_TOKEN" \
 -o beautiful_day.wav \
 https://api.daisys.ai/v1/speak/takes/t01haybgb16dn9dk0p5je47qz74/wav

   In the above, we retrieve a .wav file and write it to disk as
   beautiful_day.wav. Note that the -L flag must be provided since the file
   is returned through a 307 redirect.

   The resulting file beautiful_day.wav can be played using command line
   programs like aplay on Linux, or any audio player such as the excellent
   VLC. You can integrate the results into your creative projects!

   It is also possible to retrieve the audio in other formats: mp3, flac,
   webm, and m4a, by retrieving at the corresponding URL,
   ../speak/takes/t01haybgb16dn9dk0p5je47qz74/mp3, etc.

Daisys API examples

   The following examples can be used to see how the Daisys API client
   library for Python can be used.

   Running examples

 export DAISYS_EMAIL=user@example.com DAISYS_PASSWORD='<my password>'
 python3 -m venv venv
 . venv/bin/activate
 python3 -m pip install daisys
 python3 -m daisys.examples.hello_daisys

  Example: Hello Daisys, synchronous client

   This example shows:

    1. How to create the synchronous client using a context manager.

    2. Get a list of voices.

    3. If there are none, how to generate a voice.

    4. Reference the voice to generate audio (a “take”) for some text.

    5. Download the resulting audio.

   Example output

 $ python3 -m examples.hello_daisys
 Found Daisys Speak API version=1 minor=0
 Found voices: []
 Not enough voices!
 Using model "shakespeare"
 Generating a voice.
 Sally speaking!
 Read 198700 bytes of wav data, wrote "hello_daisys.wav".
 Checking take: True
 Checking list of takes: True
 Deleting take t01hbbgw0zz4e9y6pb9qdxnrmag: True
 Deleting voice v01hbbgtrvk50pxwyjvvsxygbza: True

   examples/hello_daisys.py

 import os, asyncio, json
 from daisys import DaisysAPI
 from daisys.v1.speak import VoiceGender, SimpleProsody, DaisysTakeGenerateError, HTTPStatusError

 # Override DAISYS_EMAIL and DAISYS_PASSWORD with your details!
 EMAIL = os.environ.get('DAISYS_EMAIL', 'user@example.com')
 PASSWORD = os.environ.get('DAISYS_PASSWORD', 'pw')

 # Please see tokens_example.py for how to use an access token instead of a password.

 def load_tokens():
     """A function to access and refresh tokens from a local file.  In practice you might
     store this somewhere more global like in a database, to re-use between sessions."""
     try:
         with open('daisys_tokens.json') as tokens_file:
             tokens = json.load(tokens_file)
             print('Loaded tokens from "daisys_tokens.json".')
             return tokens['access_token'], tokens['refresh_token']
     except (FileNotFoundError, json.JSONDecodeError):
         return None, None

 ACCESS_TOKEN, REFRESH_TOKEN = load_tokens()

 def main():
     with DaisysAPI('speak', email=EMAIL, password=PASSWORD) as speak:
         print('Found Daisys Speak API', speak.version())

         # The following is an example of how to use the Daisys API for generating a voice
         # and then using it in a speech generation task.  The API generates "takes"
         # representing one or more sentences from a speaker.  The same example is possible
         # with the synchronous client, where the 'await' keywords should be removed.

         # Get a list of all voices
         voices = speak.get_voices()
         print('Found voices:', [voice.name for voice in voices])

         # Choose one
         if len(voices) > 0:
             voice = voices[-1]
             delete_voice = False
         else:
             print('Not enough voices!')

             # Okay, let's generate a voice.

             # First we need to know the model.
             models = speak.get_models()
             if len(models) > 0:
                 model = models[0]
                 print(f'Using model "{model.displayname}"')
             else:
                 print('No models found!')
                 return

             print('Generating a voice.')
             voice = speak.generate_voice(name='Lucy', gender=VoiceGender.FEMALE, model=model.name)
             delete_voice = True

             # Try to modify the voice's name
             voice.name = 'Sally'
             speak.update_voice(**dict(voice))
             voice = speak.get_voice(voice.voice_id)

         # Now we have a voice.
         print(voice.name, 'speaking!')

         try:
             # Synthesize some audio from text
             take = speak.generate_take(voice_id=voice.voice_id, text="Hello there, I am Daisys!",
                                        prosody=SimpleProsody(pace=-3, pitch=2, expression=10))
         except DaisysTakeGenerateError as e:
             print('Error generating take:', str(e))
             return

         # The take is now READY.  We get its associated audio file.  We provide a filename
         # so that it gets written to disk, but it is also returned.
         audio_wav = speak.get_take_audio(take.take_id, file='hello_daisys.mp3', format='mp3')

         print(f'Read {len(audio_wav)} bytes of wav data, wrote "hello_daisys.wav".')

         # Let's check if we can get info on it again.
         check_take = speak.get_take(take.take_id)
         print('Checking take:', check_take == take)

         # Let's check if we can find it in the most recent 5 takes.
         last_5_takes = speak.get_takes(length=5)
         print('Checking list of takes:', take.take_id in [t.take_id for t in last_5_takes])

         # Delete the take
         print(f'Deleting take {take.take_id}:', speak.delete_take(take.take_id))

         # Delete the voice
         if delete_voice:
             print(f'Deleting voice {voice.voice_id}:', speak.delete_voice(voice.voice_id))

 if __name__=='__main__':
     try:
         main()
     except HTTPStatusError as e:
         try:
             print(f'HTTP error status {e.response.status_code}: {e.response.json()["detail"]}, {e.request.url}')
         except:
             print(f'HTTP error status {e.response.status_code}: {e.response.text}, {e.request.url}')

  Example: Hello Daisys, asynchronous client

   This example shows:

    1. How to create the asyncio client using a context manager.

    2. Get a list of voices.

    3. If there are none, how to generate a voice.

    4. Reference the voice to generate audio (a “take”) for some text.

    5. Download the resulting audio.

    6. Get the take information by identifier or as a filtered list.

   To run it, you must replace the username and password with your
   credentials.

   Example output

 $ python3 -m examples.hello_daisys_async
 Found Daisys Speak API version=1 minor=0
 Found voices: []
 Not enough voices!
 Using model "shakespeare"
 Generating a voice.
 Sally speaking!
 Read 208940 bytes of wav data, wrote "hello_daisys.wav".
 Checking take: True
 Checking list of takes: True
 Deleting take t01hbbgyx2008ggp61pzh6jaemf: True
 Deleting voice v01hbbgyrpxxbcj6q37f1yd03gd: True

   examples/hello_daisys_async.py

 import os, asyncio
 from daisys import DaisysAPI
 from daisys.v1.speak import VoiceGender, SimpleProsody, DaisysTakeGenerateError, HTTPStatusError

 # Override DAISYS_EMAIL and DAISYS_PASSWORD with your details!
 EMAIL = os.environ.get('DAISYS_EMAIL', 'user@example.com')
 PASSWORD = os.environ.get('DAISYS_PASSWORD', 'pw')

 # Please see tokens_example.py for how to use an access token instead of a password.

 async def main():
     async with DaisysAPI('speak', email=EMAIL, password=PASSWORD) as speak:
         print('Found Daisys Speak API', await speak.version())

         # The following is an example of how to use the Daisys API for generating a voice
         # and then using it in a speech generation task.  The API generates "takes"
         # representing one or more sentences from a speaker.  The same example is possible
         # with the synchronous client, where the 'await' keywords should be removed.

         # Get a list of all voices
         voices = await speak.get_voices()
         print('Found voices:', [voice.name for voice in voices])

         # Choose one
         if len(voices) > 0:
             voice = voices[-1]
             delete_voice = False
         else:
             print('Not enough voices!')

             # Okay, let's generate a voice.

             # First we need to know the model.
             models = await speak.get_models()
             if len(models) > 0:
                 model = models[0]
                 print(f'Using model "{model.displayname}"')
             else:
                 print('No models found!')
                 return

             print('Generating a voice.')
             voice = await speak.generate_voice(name='Lucy', gender=VoiceGender.FEMALE, model=model.name)
             delete_voice = True

             # Try to modify the voice's name
             voice.name = 'Sally'
             await speak.update_voice(**dict(voice))
             voice = await speak.get_voice(voice.voice_id)

         # Now we have a voice.
         print(voice.name, 'speaking!')

         try:
             # Synthesize some audio from text
             take = await speak.generate_take(voice_id=voice.voice_id, text="Hello there, I am Daisys!",
                                              prosody=SimpleProsody(pace=-3, pitch=2, expression=10))
         except DaisysTakeGenerateError as e:
             print('Error generating take:', str(e))
             return

         # The take is now READY.  We get its associated audio file.  We provide a filename
         # so that it gets written to disk, but it is also returned.
         audio_wav = await speak.get_take_audio(take.take_id, file='hello_daisys.wav')

         print(f'Read {len(audio_wav)} bytes of wav data, wrote "hello_daisys.wav".')

         # Let's check if we can get info on it again.
         check_take = await speak.get_take(take.take_id)
         print('Checking take:', check_take == take)

         # Let's check if we can find it in the most recent 5 takes.
         last_5_takes = await speak.get_takes(length=5)
         print('Checking list of takes:', take.take_id in [t.take_id for t in last_5_takes])

         # Delete the take
         print(f'Deleting take {take.take_id}:', await speak.delete_take(take.take_id))

         # Delete the voice
         if delete_voice:
             print(f'Deleting voice {voice.voice_id}:', await speak.delete_voice(voice.voice_id))

 if __name__=='__main__':
     try:
         asyncio.run(main())
     except HTTPStatusError as e:
         try:
             print(f'HTTP error status {e.response.status_code}: {e.response.json()["detail"]}')
         except:
             print(f'HTTP error status {e.response.status_code}: {e.response.text}')

  Daisys API websocket examples

   In addition to retrieving audio by means of the REST API (see Retrieving
   audio), takes can also be requested and audio can be streamed using a
   websocket connection. Do note that if the /wav endpoint is accessed before
   a take is “ready”, the .wav file will be streamed while it is generated,
   so the complexities of the websocket connection may not be necessary,
   depending on your application. However, the websocket connection does
   provide the lowest latency since the connection is directly made to a
   worker node.

   On the other hand since a single shared connection is used, requests over
   websocket are essentially serialized. For batch-style jobs where
   throughput rather than latency is a concern, the REST API is encouraged,
   since it distributes the jobs over multiple workers.

   The websocket streaming is also higher complexity than using the REST API.
   A detailed definition of the websocket streaming protocol can be found at
   Daisys API websockets.

    Parts vs chunks

   There are two streaming modes: “parts” and “chunks”, each example by
   default shows “parts” mode but can be put in “chunks” mode by executing
   with an argument --chunks.

   The difference is:

     * For audio generation, an input paragraph or document is broken up into
       multiple parts that end in silence, usually corresponding with a
       sentence.

     * In “parts” streaming mode, the default, each part is sent in a
       separate message. Each part contains a wav header. The intention is
       that this can be parsed and played directly by an audio player, and
       each part can be sequenced one after the other.

     * In “chunks” streaming mode, the parts similarly are each composed of a
       wav file with a header, however the file is sent in small chunks as it
       is generated. This results in reduced latency, at the expense that the
       chunks must be combined on reception, either by feeding them into an
       audio stream or concatenating them into a final wav file. Only the
       first chunk contains the wav header, and the length it indicates
       corresponds to the full part.

   Furthermore each request results in a stream of both text and binary
   messages. The former contain the entire contents of the take’s
   TakeResponse structure, and is transmitted whenever the status field
   changes. The latter contains the audio parts or chunks.

    Callbacks vs iterator

   for both the “parts” and “chunks” mode, the Python API provides either a
   callback-based mechanism for receiving status and audio messages, as well
   as a wrapper that provides an iterator-style interface called
   iter_request() to the same information which tends to simplify client
   code, see the corresponding examples for synchronous and async clients.

   When callbacks or iterators are executed, order has already been
   reconstructed, so parts and chunks are delivered to the user code in the
   correct order.

     Due to the nature of websocket connections, the order of incoming
     messages is not guaranteed. This is why the part_id and chunk_id values
     are included in all messages (chunk_id only if “chunks” streaming option
     is specified), so that the correct order can be reconstructed in the
     receiving client. This is also taken care of by the Python library, and
     is demonstrated for JavaScript in the websocket_client web app example.

    Fetching the websocket URL directly

   As mentioned, the last example websocket_client shows how to integrate
   websockets into a web app, and therefore in this case the stream ingestion
   is performed by JavaScript. Therefore the Python library is only used to
   retrieve the websocket URL using websocket_url() (which includes a
   lifetime-limited secret that is distinct from your access token for
   security) and the same work described above of connecting to the
   websocket, sending requests, and iterating over the ordered incoming
   status and audio messages is performed by the included JavaScript code.

      Example: Websocket example, synchronous client

   This example shows:

    1. How to open a websocket connection using a context manager.

    2. Generate a take, specifying status and audio callbacks.

    3. The signature of each of these callbacks and how to interpret their
       arguments.

    4. How to make a request with and without chunks enabled. (Add argument
       --chunks.)

   Example output

 $ python3 -m examples.websocket_example
 Found Daisys Speak API version=1 minor=0
 Status.WAITING
 Status.STARTED
 [0.739s] Received part_id=0 (chunk_id=None) for take_id='t01jqrjc9bfx8z0w6zarf8hcq8y' with audio length 235564
 Read 235564 bytes of wav data, wrote "websocket_part1.wav".
 Status.PROGRESS_50
 [1.166s] Received part_id=1 (chunk_id=None) for take_id='t01jqrjc9bfx8z0w6zarf8hcq8y' with audio length 106540
 Read 106540 bytes of wav data, wrote "websocket_part2.wav".
 [1.166s] Received part_id=2 (chunk_id=None) for take_id='t01jqrjc9bfx8z0w6zarf8hcq8y' with audio length (empty -- done receiving)
 Status.READY
 Deleting take t01jqrjc9bfx8z0w6zarf8hcq8y: True

   Example output (chunks enabled)

 $ python3 -m examples.websocket_example --chunks
 Found Daisys Speak API version=1 minor=0
 Status.WAITING
 Status.STARTED
 [0.311s] Received part_id=0 (chunk_id=0) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4140
 [0.323s] Received part_id=0 (chunk_id=1) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [0.335s] Received part_id=0 (chunk_id=2) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [0.347s] Received part_id=0 (chunk_id=3) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [0.359s] Received part_id=0 (chunk_id=4) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [0.372s] Received part_id=0 (chunk_id=5) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [0.385s] Received part_id=0 (chunk_id=6) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [0.396s] Received part_id=0 (chunk_id=7) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [0.409s] Received part_id=0 (chunk_id=8) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [0.421s] Received part_id=0 (chunk_id=9) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [0.433s] Received part_id=0 (chunk_id=10) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [0.445s] Received part_id=0 (chunk_id=11) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [0.457s] Received part_id=0 (chunk_id=12) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [0.469s] Received part_id=0 (chunk_id=13) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [0.479s] Received part_id=0 (chunk_id=14) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [0.489s] Received part_id=0 (chunk_id=15) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [0.500s] Received part_id=0 (chunk_id=16) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [0.510s] Received part_id=0 (chunk_id=17) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [0.520s] Received part_id=0 (chunk_id=18) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [0.530s] Received part_id=0 (chunk_id=19) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [0.540s] Received part_id=0 (chunk_id=20) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [0.550s] Received part_id=0 (chunk_id=21) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [0.560s] Received part_id=0 (chunk_id=22) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [0.570s] Received part_id=0 (chunk_id=23) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [0.580s] Received part_id=0 (chunk_id=24) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [0.590s] Received part_id=0 (chunk_id=25) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [0.600s] Received part_id=0 (chunk_id=26) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [0.610s] Received part_id=0 (chunk_id=27) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [0.621s] Received part_id=0 (chunk_id=28) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [0.631s] Received part_id=0 (chunk_id=29) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 2560
 [0.631s] Received part_id=0 (chunk_id=30) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length (empty -- done receiving)
 Read 121388 bytes of wav data, wrote "websocket_part1.wav".
 Status.PROGRESS_50
 [0.979s] Received part_id=1 (chunk_id=0) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4140
 [0.989s] Received part_id=1 (chunk_id=1) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [0.998s] Received part_id=1 (chunk_id=2) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [1.007s] Received part_id=1 (chunk_id=3) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [1.018s] Received part_id=1 (chunk_id=4) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [1.028s] Received part_id=1 (chunk_id=5) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [1.038s] Received part_id=1 (chunk_id=6) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [1.048s] Received part_id=1 (chunk_id=7) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [1.058s] Received part_id=1 (chunk_id=8) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [1.069s] Received part_id=1 (chunk_id=9) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [1.079s] Received part_id=1 (chunk_id=10) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [1.089s] Received part_id=1 (chunk_id=11) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [1.100s] Received part_id=1 (chunk_id=12) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 4096
 [1.109s] Received part_id=1 (chunk_id=13) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length 2048
 [1.110s] Received part_id=1 (chunk_id=14) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length (empty -- done receiving)
 Read 55340 bytes of wav data, wrote "websocket_part2.wav".
 [1.110s] Received part_id=2 (chunk_id=0) for take_id='t01jqrjq6xs1vbkdm1493e60gv2' with audio length (empty -- done receiving)
 Status.READY
 Deleting take t01jqrjq6xs1vbkdm1493e60gv2: True

   examples/websocket_example.py

 import sys, os, time
 from typing import Optional
 from daisys import DaisysAPI
 from daisys.v1.speak import (DaisysWebsocketGenerateError, HTTPStatusError, Status, TakeResponse,
                              StreamOptions, StreamMode)

 # Override DAISYS_EMAIL and DAISYS_PASSWORD with your details!
 EMAIL = os.environ.get('DAISYS_EMAIL', 'user@example.com')
 PASSWORD = os.environ.get('DAISYS_PASSWORD', 'pw')

 # Please see tokens_example.py for how to use an access token instead of a password.

 def main(chunks):
     with DaisysAPI('speak', email=EMAIL, password=PASSWORD) as speak:
         print('Found Daisys Speak API', speak.version())

         # A buffer to receive parts; we initialize with a single empty bytes()
         # because we will use it to accumulate chunks of the current wav file
         # there.  In total we will end with a list of wav files, one for each
         # part.  Parts are bits of speech, usually full sentences, that end with
         # silence.
         audio_wavs = [bytes()]

         # Assume at least one voice is available
         voice = speak.get_voices()[0]

         with speak.websocket(voice_id=voice.voice_id) as ws:
             # Flags we can use to only wait on our one take request; we wait
             # until the take is READY, and we also wait until we are done
             # receive all audio parts.
             done = False
             ready = False

             # Time the latency from when we submit the request until each part
             # is received.
             t0 = time.time()

             # The audio callback receives "parts" consisting of audio .wav files
             # with WAV headers on each part.  Depending on the stream settings,
             # the file may be divided into chunks, where chunk_id==None indicates
             # the last chunk of a part.  If audio==None, then no more parts will
             # arrive for that take_id.
             def audio_cb(request_id: int, take_id: str, part_id: int, chunk_id: Optional[int],
                          audio: Optional[bytes]):
                 nonlocal done

                 # Report timing info and function arguments
                 print(f'[{time.time()-t0:0.3f}s] Received {part_id=} ({chunk_id=}) for {take_id=} '
                       'with audio length', len(audio) if audio else '(empty -- done receiving)')

                 # We only requested one take_id; the take_id is generated by the
                 # Daisys API, so we do not know it until the first status
                 # message arrives.  Therefore we can check that the request_id
                 # is the expected one.
                 assert request_id == generate_request_id
                 assert generated_take is None or  take_id == generated_take.take_id

                 if audio is None:
                     # If stream is done for this part
                     if chunk_id in [0, None]:
                         # If we have any audio data, write out the last file
                         if len(audio_wavs[-1]) > 0:
                             with open(f'websocket_part{len(audio_wavs)}.wav', 'wb') as f:
                                 f.write(audio_wavs[-1])
                                 print(f'Read {len(audio_wavs[-1])} bytes of wav data, wrote "{f.name}".')
                         # Flag that we are done receiving audio
                         done = True

                     # If we are receiving the last chunk of a part
                     elif chunk_id > 0:
                         # Write out the part
                         with open(f'websocket_part{len(audio_wavs)}.wav', 'wb') as f:
                             f.write(audio_wavs[-1])
                             print(f'Read {len(audio_wavs[-1])} bytes of wav data, wrote "{f.name}".')

                         # Start a new part
                         audio_wavs.append(bytes())

                 # Otherwise append the chunk.
                 else:
                     audio_wavs[-1] = audio_wavs[-1] + audio

                     # If non-chunked stream, the part is ended immediately
                     if chunk_id is None:
                         # If we have any audio data, write out the file
                         with open(f'websocket_part{len(audio_wavs)}.wav', 'wb') as f:
                             f.write(audio_wavs[-1])
                             print(f'Read {len(audio_wavs[-1])} bytes of wav data, wrote "{f.name}".')

                         # Start a new part
                         audio_wavs.append(bytes())

             # The status callback is called every time the take's status
             # changes.  Here we use it to end the update loop.
             def status_cb(request_id: int, take: TakeResponse):
                 nonlocal ready, generated_take
                 assert request_id == generate_request_id
                 generated_take = take
                 print(take.status)
                 if take.status == Status.READY:
                     ready = True

             # Submit a request to generate a take over the websocket connection.
             generate_request_id = ws.generate_take(
                 voice_id=voice.voice_id,
                 text='Hello from Daisys websockets! How may I help you?',
                 status_callback=status_cb,
                 audio_callback=audio_cb,

                 # Optional
                 stream_options=StreamOptions(mode=StreamMode.CHUNKS) if chunks else None,
             )

             # Will be filled in by callbacks. On submitting the generate
             # request, we do not yet know what take_id will be assigned so we
             # must discover it by means of the status callback.
             generated_take = None

             # We loop on the websocket while waiting 5 seconds between updates,
             # and end when the take as been set to READY and all audio has been
             # received. This update waits 1 second by default, here we set to 5
             # seconds, but it can also wait forever by setting timeout to None
             # or be made a non-blocking operation by setting timeout to 0.
             # (Important: in async client, timeout=0 leads to TimeoutError, it
             # cannot be used for non-blocking operations with asyncio.)
             while not (ready and done) and (time.time() - t0) < 60:
                 try:
                     ws.update(timeout=5)
                 except DaisysWebsocketGenerateError as e:
                     # As opposed to other websocket errors, if a generate error
                     # occurs it does not necessarily mean we want to close the
                     # stream.
                     print(e)

                     # In this example, however, we actually do, because we only
                     # requested a single take, so stop here.
                     break

         # Delete the take
         if generated_take:
             print(f'Deleting take {generated_take.take_id}:', speak.delete_take(generated_take.take_id))

 if __name__=='__main__':
     try:
         main('--chunks' in sys.argv[1:])
     except HTTPStatusError as e:
         try:
             print(f'HTTP error status {e.response.status_code}: {e.response.json()["detail"]}, {e.request.url}')
         except:
             print(f'HTTP error status {e.response.status_code}: {e.response.text}, {e.request.url}')

      Example: Websocket example, synchronous client with iterator

   This example shows:

    1. How to open a websocket connection using a context manager.

    2. Generate a take.

    3. How to iterate over the resulting status and audio messages using
       iter_request().

    4. How to make a request with and without chunks enabled. (Add argument
       --chunks.)

   Example output

 $ python3 -m examples.websocket_example_iter
 Found Daisys Speak API version=1 minor=0
 [0.002] Take status was changed to: WAITING.
 [0.022] Take status was changed to: STARTED.
 [0.748] New part being received.
 [0.748] Received audio chunk of size 233472.
 [1.204] Take status was changed to: PROGRESS_50.
 [1.208] New part being received.
 [1.208] Received audio chunk of size 116736.
 [2.597] Take status was changed to: READY.
 Deleting take t01jqrj9j7hyx49enqya9qeas3t: True

   Example output (chunks enabled)

 $ python3 -m examples.websocket_example_iter --chunks
 Found Daisys Speak API version=1 minor=0
 [0.002] Take status was changed to: WAITING.
 [0.026] Take status was changed to: STARTED.
 [0.314] New part being received.
 [0.314] Received audio chunk of size 4096.
 [0.328] Received audio chunk of size 4096.
 [0.341] Received audio chunk of size 4096.
 [0.351] Received audio chunk of size 4096.
 [0.361] Received audio chunk of size 4096.
 [0.371] Received audio chunk of size 4096.
 [0.381] Received audio chunk of size 4096.
 [0.391] Received audio chunk of size 4096.
 [0.401] Received audio chunk of size 4096.
 [0.411] Received audio chunk of size 4096.
 [0.421] Received audio chunk of size 4096.
 [0.431] Received audio chunk of size 4096.
 [0.442] Received audio chunk of size 4096.
 [0.452] Received audio chunk of size 4096.
 [0.462] Received audio chunk of size 4096.
 [0.472] Received audio chunk of size 4096.
 [0.482] Received audio chunk of size 4096.
 [0.492] Received audio chunk of size 4096.
 [0.502] Received audio chunk of size 4096.
 [0.512] Received audio chunk of size 4096.
 [0.521] Received audio chunk of size 4096.
 [0.532] Received audio chunk of size 4096.
 [0.542] Received audio chunk of size 4096.
 [0.551] Received audio chunk of size 4096.
 [0.561] Received audio chunk of size 4096.
 [0.572] Received audio chunk of size 4096.
 [0.582] Received audio chunk of size 4096.
 [0.592] Received audio chunk of size 4096.
 [0.603] Received audio chunk of size 4096.
 [0.613] Received audio chunk of size 1536.
 [0.963] Take status was changed to: PROGRESS_50.
 [0.966] New part being received.
 [0.966] Received audio chunk of size 4096.
 [0.976] Received audio chunk of size 4096.
 [0.985] Received audio chunk of size 4096.
 [0.994] Received audio chunk of size 4096.
 [1.004] Received audio chunk of size 4096.
 [1.014] Received audio chunk of size 4096.
 [1.024] Received audio chunk of size 4096.
 [1.034] Received audio chunk of size 4096.
 [1.044] Received audio chunk of size 4096.
 [1.055] Received audio chunk of size 4096.
 [1.065] Received audio chunk of size 4096.
 [1.075] Received audio chunk of size 4096.
 [1.085] Received audio chunk of size 4096.
 [1.095] Received audio chunk of size 3072.
 [2.600] Take status was changed to: READY.
 Deleting take t01jqrjxc257e1mr0r0z65ak4qb: True

   examples/websocket_example_async_iter.py

 import sys, os, asyncio, time
 from typing import Optional
 from daisys import DaisysAPI
 from daisys.v1.speak import (DaisysWebsocketGenerateError, HTTPStatusError, Status, TakeResponse,
                              StreamOptions, StreamMode)

 # Override DAISYS_EMAIL and DAISYS_PASSWORD with your details!
 EMAIL = os.environ.get('DAISYS_EMAIL', 'user@example.com')
 PASSWORD = os.environ.get('DAISYS_PASSWORD', 'pw')

 # Please see tokens_example.py for how to use an access token instead of a password.

 async def main(chunks):
     async with DaisysAPI('speak', email=EMAIL, password=PASSWORD) as speak:
         print('Found Daisys Speak API', await speak.version())

         # A buffer to receive parts; we initialize with a single empty bytes()
         # because we will use it to accumulate chunks of the current wav file
         # there.  In total we will end with a list of wav files, one for each
         # part.  Parts are bits of speech, usually full sentences, that end with
         # silence.
         audio_wavs = [bytes()]

         # Assume at least one voice is available
         voice = (await speak.get_voices())[0]

         async with speak.websocket(voice_id=voice.voice_id) as ws:
             # Time the latency from when we submit the request until each part
             # is received.
             t0 = time.time()

             # Submit a request to generate a take over the websocket connection.
             generate_request_id = await ws.generate_take(
                 voice_id=voice.voice_id,
                 text='Hello from Daisys websockets! How may I help you?',

                 # Optional
                 stream_options=StreamOptions(mode=StreamMode.CHUNKS) if chunks else None,
             )

             # The use of an interator simplifies streaming, here we show how to
             # get both status and audio chunks from the same iterator.
             async for take_id, take, header, audio in ws.iter_request(generate_request_id):
                 now = time.time() - t0
                 if take is not None:
                     print(f'[{now:.03f}] Take status was changed to: {take.status.name}.')
                 if header is not None:
                     print(f'[{now:.03f}] New part being received.')
                 if audio is not None:
                     print(f'[{now:.03f}] Received audio chunk of size {len(audio)}.')

         # Delete the take
         if take_id:
             print(f'Deleting take {take_id}:', await speak.delete_take(take_id))

 if __name__=='__main__':
     try:
         asyncio.run(main('--chunks' in sys.argv[1:]))
     except HTTPStatusError as e:
         try:
             print(f'HTTP error status {e.response.status_code}: {e.response.json()["detail"]}, {e.request.url}')
         except:
             print(f'HTTP error status {e.response.status_code}: {e.response.text}, {e.request.url}')

      Example: Websocket example, asynchronous client

   This example shows:

    1. How to open a websocket connection using an async context manager.

    2. Generate a take, specifying status and audio callbacks.

    3. The signature of each of these callbacks and how to interpret their
       arguments.

    4. How to make a request with and without chunks enabled. (Add argument
       --chunks.)

   Example output

 $ python3 -m examples.websocket_example_async
 Found Daisys Speak API version=1 minor=0
 Status.WAITING
 Status.STARTED
 [0.751s] Received part_id=0 (chunk_id=None) for take_id='t01jqrjedpjb11hpg40h9kkydpk' with audio length 245804
 appending audio 245804
 Read 245804 bytes of wav data, wrote "websocket_part1.wav".
 Status.PROGRESS_50
 [1.183s] Received part_id=1 (chunk_id=None) for take_id='t01jqrjedpjb11hpg40h9kkydpk' with audio length 112684
 appending audio 112684
 Read 112684 bytes of wav data, wrote "websocket_part2.wav".
 [1.184s] Received part_id=2 (chunk_id=None) for take_id='t01jqrjedpjb11hpg40h9kkydpk' with audio length (empty -- done receiving)
 stream done
 Status.READY
 Deleting take t01jqrjedpjb11hpg40h9kkydpk: True

   Example output (chunks enabled)

 $ python3 -m examples.websocket_example_async --chunks
 Found Daisys Speak API version=1 minor=0
 Status.WAITING
 Status.STARTED
 [0.311s] Received part_id=0 (chunk_id=0) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4140
 appending audio 4140
 [0.324s] Received part_id=0 (chunk_id=1) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 8236
 [0.338s] Received part_id=0 (chunk_id=2) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 12332
 [0.351s] Received part_id=0 (chunk_id=3) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 16428
 [0.365s] Received part_id=0 (chunk_id=4) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 20524
 [0.378s] Received part_id=0 (chunk_id=5) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 24620
 [0.389s] Received part_id=0 (chunk_id=6) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 28716
 [0.399s] Received part_id=0 (chunk_id=7) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 32812
 [0.409s] Received part_id=0 (chunk_id=8) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 36908
 [0.419s] Received part_id=0 (chunk_id=9) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 41004
 [0.429s] Received part_id=0 (chunk_id=10) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 45100
 [0.439s] Received part_id=0 (chunk_id=11) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 49196
 [0.449s] Received part_id=0 (chunk_id=12) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 53292
 [0.459s] Received part_id=0 (chunk_id=13) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 57388
 [0.469s] Received part_id=0 (chunk_id=14) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 61484
 [0.479s] Received part_id=0 (chunk_id=15) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 65580
 [0.489s] Received part_id=0 (chunk_id=16) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 69676
 [0.500s] Received part_id=0 (chunk_id=17) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 73772
 [0.510s] Received part_id=0 (chunk_id=18) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 77868
 [0.520s] Received part_id=0 (chunk_id=19) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 81964
 [0.530s] Received part_id=0 (chunk_id=20) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 86060
 [0.540s] Received part_id=0 (chunk_id=21) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 90156
 [0.550s] Received part_id=0 (chunk_id=22) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 94252
 [0.560s] Received part_id=0 (chunk_id=23) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 98348
 [0.570s] Received part_id=0 (chunk_id=24) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 102444
 [0.580s] Received part_id=0 (chunk_id=25) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 106540
 [0.590s] Received part_id=0 (chunk_id=26) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 110636
 [0.600s] Received part_id=0 (chunk_id=27) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 114732
 [0.610s] Received part_id=0 (chunk_id=28) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 118828
 [0.620s] Received part_id=0 (chunk_id=29) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 2048
 appending audio 120876
 [0.621s] Received part_id=0 (chunk_id=30) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length (empty -- done receiving)
 part done
 Read 120876 bytes of wav data, wrote "websocket_part1.wav".
 Status.PROGRESS_50
 [0.976s] Received part_id=1 (chunk_id=0) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4140
 appending audio 4140
 [0.985s] Received part_id=1 (chunk_id=1) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 8236
 [0.995s] Received part_id=1 (chunk_id=2) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 12332
 [1.004s] Received part_id=1 (chunk_id=3) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 16428
 [1.014s] Received part_id=1 (chunk_id=4) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 20524
 [1.024s] Received part_id=1 (chunk_id=5) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 24620
 [1.034s] Received part_id=1 (chunk_id=6) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 28716
 [1.044s] Received part_id=1 (chunk_id=7) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 32812
 [1.054s] Received part_id=1 (chunk_id=8) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 36908
 [1.064s] Received part_id=1 (chunk_id=9) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 41004
 [1.074s] Received part_id=1 (chunk_id=10) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 45100
 [1.084s] Received part_id=1 (chunk_id=11) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 49196
 [1.095s] Received part_id=1 (chunk_id=12) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 4096
 appending audio 53292
 [1.105s] Received part_id=1 (chunk_id=13) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length 1024
 appending audio 54316
 [1.105s] Received part_id=1 (chunk_id=14) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length (empty -- done receiving)
 part done
 Read 54316 bytes of wav data, wrote "websocket_part2.wav".
 [1.105s] Received part_id=2 (chunk_id=0) for take_id='t01jqrjh3yrbzpd79q1nprcrbjy' with audio length (empty -- done receiving)
 stream done
 Status.READY
 Deleting take t01jqrjh3yrbzpd79q1nprcrbjy: True

   examples/websocket_example_async.py

 import sys, os, asyncio, time
 from typing import Optional
 from daisys import DaisysAPI
 from daisys.v1.speak import (DaisysWebsocketGenerateError, HTTPStatusError, Status, TakeResponse,
                              StreamOptions, StreamMode)

 # Override DAISYS_EMAIL and DAISYS_PASSWORD with your details!
 EMAIL = os.environ.get('DAISYS_EMAIL', 'user@example.com')
 PASSWORD = os.environ.get('DAISYS_PASSWORD', 'pw')

 # Please see tokens_example.py for how to use an access token instead of a password.


 async def main(chunks):
     async with DaisysAPI('speak', email=EMAIL, password=PASSWORD) as speak:
         print('Found Daisys Speak API', await speak.version())

         # A buffer to receive parts; we initialize with a single empty bytes()
         # because we will use it to accumulate chunks of the current wav file
         # there.  In total we will end with a list of wav files, one for each
         # part.  Parts are bits of speech, usually full sentences, that end with
         # silence.
         audio_wavs = [bytes()]

         # Assume at least one voice is available
         voice = (await speak.get_voices())[0]

         async with speak.websocket(voice_id=voice.voice_id) as ws:
             # Flags we can use to only wait on our one take request; we wait
             # until the take is READY, and we also wait until we are done
             # receive all audio parts.
             done = False
             ready = False

             # Time the latency from when we submit the request until each part
             # is received.
             t0 = time.time()

             # The audio callback receives "parts" consisting of audio .wav files
             # with WAV headers on each part.  Depending on the stream settings,
             # the file may be divided into chunks, where chunk_id==None indicates
             # the last chunk of a part.  If audio==None, then no more parts will
             # arrive for that take_id.
             async def audio_cb(request_id: int, take_id: str, part_id: int,
                                chunk_id: int|None, audio: bytes|None):
                 nonlocal done

                 # Report timing info and function arguments
                 print(f'[{time.time()-t0:0.3f}s] Received {part_id=} ({chunk_id=}) for {take_id=} '
                       'with audio length', len(audio) if audio else '(empty -- done receiving)')

                 # We only requested one take_id; the take_id is generated by the
                 # Daisys API, so we do not know it until the first status
                 # message arrives.  Therefore we can check that the request_id
                 # is the expected one.
                 assert request_id == generate_request_id
                 assert generated_take is None or  take_id == generated_take.take_id

                 if audio is None:
                     # If stream is done for this part
                     if chunk_id in [0, None]:
                         print('stream done')
                         # If we have any audio data, write out the last file
                         if len(audio_wavs[-1]) > 0:
                             with open(f'websocket_part{len(audio_wavs)}.wav', 'wb') as f:
                                 f.write(audio_wavs[-1])
                                 print(f'Read {len(audio_wavs[-1])} bytes of wav data, wrote "{f.name}".')
                         # Flag that we are done receiving audio
                         done = True

                     # If we are receiving the last chunk of a part
                     elif chunk_id > 0:
                         print('part done')
                         # Write out the part
                         with open(f'websocket_part{len(audio_wavs)}.wav', 'wb') as f:
                             f.write(audio_wavs[-1])
                             print(f'Read {len(audio_wavs[-1])} bytes of wav data, wrote "{f.name}".')

                         # Start a new part
                         audio_wavs.append(bytes())

                 # Otherwise append the chunk.
                 else:
                     audio_wavs[-1] = audio_wavs[-1] + audio
                     print('appending audio', len(audio_wavs[-1]))

                     # If non-chunked stream, the part is ended immediately
                     if chunk_id is None:
                         # If we have any audio data, write out the file
                         with open(f'websocket_part{len(audio_wavs)}.wav', 'wb') as f:
                             f.write(audio_wavs[-1])
                             print(f'Read {len(audio_wavs[-1])} bytes of wav data, wrote "{f.name}".')

                         # Start a new part
                         audio_wavs.append(bytes())

             # The status callback is called every time the take's status
             # changes.  Here we use it to end the update loop.
             async def status_cb(request_id: int, take: TakeResponse):
                 nonlocal ready, generated_take
                 assert request_id == generate_request_id
                 generated_take = take
                 print(take.status)
                 if take.status == Status.READY:
                     ready = True

             # Submit a request to generate a take over the websocket connection.
             generate_request_id = await ws.generate_take(
                 voice_id=voice.voice_id,
                 text='Hello from Daisys websockets! How may I help you?',
                 status_callback=status_cb,
                 audio_callback=audio_cb,

                 # Optional
                 stream_options=StreamOptions(mode=StreamMode.CHUNKS) if chunks else None,
             )

             # Will be filled in by callbacks. On submitting the generate
             # request, we do not yet know what take_id will be assigned so we
             # must discover it by means of the status callback.
             generated_take = None

             # We loop on the websocket while waiting 5 seconds between updates,
             # and end when the take as been set to READY and all audio has been
             # received. This update waits 1 second by default, here we set to 5
             # seconds, but it can also wait forever by setting timeout to None
             # or be made a non-blocking operation by setting timeout to 0.
             # (Important: in async client, timeout=0 leads to TimeoutError, it
             # cannot be used for non-blocking operations with asyncio.)
             while not (ready and done) and (time.time() - t0) < 60:
                 try:
                     await ws.update(timeout=5)
                 except DaisysWebsocketGenerateError as e:
                     # As opposed to other websocket errors, if a generate error
                     # occurs it does not necessarily mean we want to close the
                     # stream.
                     print(e)

                     # In this example, however, we actually do, because we only
                     # requested a single take, so stop here.
                     break

         # Delete the take
         if generated_take:
             print(f'Deleting take {generated_take.take_id}:',
                   await speak.delete_take(generated_take.take_id))

 if __name__=='__main__':
     try:
         asyncio.run(main(chunks='--chunks' in sys.argv[1:]))
     except HTTPStatusError as e:
         try:
             print(f'HTTP error status {e.response.status_code}: {e.response.json()["detail"]}, {e.request.url}')
         except:
             print(f'HTTP error status {e.response.status_code}: {e.response.text}, {e.request.url}')

      Example: Websocket example, asynchronous client with iterator

   This example shows:

    1. How to open a websocket connection using an async context manager.

    2. Generate a take.

    3. How to iterate over the resulting status and audio messages using
       iter_request().

    4. How to make a request with and without chunks enabled. (Add argument
       --chunks.)

   Example output

 $ python3 -m examples.websocket_example_async_iter
 Found Daisys Speak API version=1 minor=0
 [0.002] Take status was changed to: WAITING.
 [0.024] Take status was changed to: STARTED.
 [0.761] New part being received.
 [0.761] Received audio chunk of size 245760.
 [1.197] Take status was changed to: PROGRESS_50.
 [1.200] New part being received.
 [1.200] Received audio chunk of size 108544.
 [2.595] Take status was changed to: READY.
 Deleting take t01jqrj756qrvrqaw59zgyxpcrw: True

   Example output (chunks enabled)

 $ python3 -m examples.websocket_example_async_iter --chunks
 Found Daisys Speak API version=1 minor=0
 [0.002] Take status was changed to: WAITING.
 [0.023] Take status was changed to: STARTED.
 [0.318] New part being received.
 [0.318] Received audio chunk of size 4096.
 [0.331] Received audio chunk of size 4096.
 [0.344] Received audio chunk of size 4096.
 [0.358] Received audio chunk of size 4096.
 [0.371] Received audio chunk of size 4096.
 [0.384] Received audio chunk of size 4096.
 [0.397] Received audio chunk of size 4096.
 [0.411] Received audio chunk of size 4096.
 [0.424] Received audio chunk of size 4096.
 [0.437] Received audio chunk of size 4096.
 [0.450] Received audio chunk of size 4096.
 [0.463] Received audio chunk of size 4096.
 [0.472] Received audio chunk of size 4096.
 [0.482] Received audio chunk of size 4096.
 [0.492] Received audio chunk of size 4096.
 [0.503] Received audio chunk of size 4096.
 [0.513] Received audio chunk of size 4096.
 [0.523] Received audio chunk of size 4096.
 [0.533] Received audio chunk of size 4096.
 [0.543] Received audio chunk of size 4096.
 [0.553] Received audio chunk of size 4096.
 [0.564] Received audio chunk of size 4096.
 [0.575] Received audio chunk of size 4096.
 [0.584] Received audio chunk of size 4096.
 [0.595] Received audio chunk of size 4096.
 [0.605] Received audio chunk of size 4096.
 [0.615] Received audio chunk of size 4096.
 [0.626] Received audio chunk of size 4096.
 [0.636] Received audio chunk of size 4096.
 [0.646] Received audio chunk of size 1024.
 [1.002] Take status was changed to: PROGRESS_50.
 [1.005] New part being received.
 [1.005] Received audio chunk of size 4096.
 [1.015] Received audio chunk of size 4096.
 [1.024] Received audio chunk of size 4096.
 [1.033] Received audio chunk of size 4096.
 [1.043] Received audio chunk of size 4096.
 [1.053] Received audio chunk of size 4096.
 [1.064] Received audio chunk of size 4096.
 [1.074] Received audio chunk of size 4096.
 [1.084] Received audio chunk of size 4096.
 [1.095] Received audio chunk of size 4096.
 [1.105] Received audio chunk of size 4096.
 [1.115] Received audio chunk of size 4096.
 [1.125] Received audio chunk of size 4096.
 [1.135] Received audio chunk of size 2048.
 [2.641] Take status was changed to: READY.
 Deleting take t01jqrk04k7fhrdgs764bv6h7p1: True

   examples/websocket_example_async_iter.py

 import sys, os, asyncio, time
 from typing import Optional
 from daisys import DaisysAPI
 from daisys.v1.speak import (DaisysWebsocketGenerateError, HTTPStatusError, Status, TakeResponse,
                              StreamOptions, StreamMode)

 # Override DAISYS_EMAIL and DAISYS_PASSWORD with your details!
 EMAIL = os.environ.get('DAISYS_EMAIL', 'user@example.com')
 PASSWORD = os.environ.get('DAISYS_PASSWORD', 'pw')

 # Please see tokens_example.py for how to use an access token instead of a password.

 async def main(chunks):
     async with DaisysAPI('speak', email=EMAIL, password=PASSWORD) as speak:
         print('Found Daisys Speak API', await speak.version())

         # A buffer to receive parts; we initialize with a single empty bytes()
         # because we will use it to accumulate chunks of the current wav file
         # there.  In total we will end with a list of wav files, one for each
         # part.  Parts are bits of speech, usually full sentences, that end with
         # silence.
         audio_wavs = [bytes()]

         # Assume at least one voice is available
         voice = (await speak.get_voices())[0]

         async with speak.websocket(voice_id=voice.voice_id) as ws:
             # Time the latency from when we submit the request until each part
             # is received.
             t0 = time.time()

             # Submit a request to generate a take over the websocket connection.
             generate_request_id = await ws.generate_take(
                 voice_id=voice.voice_id,
                 text='Hello from Daisys websockets! How may I help you?',

                 # Optional
                 stream_options=StreamOptions(mode=StreamMode.CHUNKS) if chunks else None,
             )

             # The use of an interator simplifies streaming, here we show how to
             # get both status and audio chunks from the same iterator.
             async for take_id, take, header, audio in ws.iter_request(generate_request_id):
                 now = time.time() - t0
                 if take is not None:
                     print(f'[{now:.03f}] Take status was changed to: {take.status.name}.')
                 if header is not None:
                     print(f'[{now:.03f}] New part being received.')
                 if audio is not None:
                     print(f'[{now:.03f}] Received audio chunk of size {len(audio)}.')

         # Delete the take
         if take_id:
             print(f'Deleting take {take_id}:', await speak.delete_take(take_id))

 if __name__=='__main__':
     try:
         asyncio.run(main('--chunks' in sys.argv[1:]))
     except HTTPStatusError as e:
         try:
             print(f'HTTP error status {e.response.status_code}: {e.response.json()["detail"]}, {e.request.url}')
         except:
             print(f'HTTP error status {e.response.status_code}: {e.response.text}, {e.request.url}')

      Example: Websocket example, web client

   This example shows how to use the Python client library to create a
   FastAPI-based web server that performs login to the Daisys API (so that
   credentials are kept secure) and interacts with the REST API to retrieve a
   websocket URL. This URL is passed to the front-end JavaScript application
   that makes the websocket connection and makes take requests, playing back
   the audio in a streaming fashion using the Web Audio API.

   The included JavaScript shows how to:

    1. Retrieve the URL using websocket_url() and open the websocket
       connection, automatically doing so again when disconnected, see
       websocket_connector.js.

    2. Define an async iterator that simplifies handling of streams for
       different requests by transforming the callback structure into a
       simple for loop, see websocket_stream.js and usage in
       websocket_client.js.

    3. Send a request for generating a take, see websocket_client.js.

    4. Use a single handler to handle incoming status messages and audio
       messages in both “parts” and “chunks” mode, see websocket_stream.js.

    5. Respectively play the audio in a simple (parts, using audio sources)
       and more complex (chunks, using dynamic audio buffers) way using the
       Web Audio API, see part_audio_player.js and chunk_audio_player.js.

   The above consists of several files, as opposed to other examples, so
   instead of repeating the example in the documentation, the reader is
   invited to follow the code in the git repository.

   The application can be launched using:

 python3 -m examples.websocket_client

   or equivalently:

 uvicorn examples.websocket_client:app

  Example: Using an access token instead of password

   This example shows:

    1. How to initially retrieve access and refresh tokens.

    2. How to use them and store them if they change (are refreshed).

   Example output

 $ python3 -m examples.tokens_example

   examples/tokens_example.py

 import os, asyncio, json
 from daisys import DaisysAPI
 from daisys.v1.speak import VoiceGender, SimpleProsody, DaisysTakeGenerateError, HTTPStatusError

 # This example shows how to use access and refresh tokens instead of a username and email
 # address for authenticating with the Daisys API.

 # Override DAISYS_EMAIL and DAISYS_PASSWORD with your details!
 EMAIL = os.environ.get('DAISYS_EMAIL', 'user@example.com')
 PASSWORD = os.environ.get('DAISYS_PASSWORD', 'pw')

 def load_tokens():
     """A function to access and refresh tokens from a local file.  In practice you might
     store this somewhere more global like in a database, to re-use between sessions."""
     try:
         with open('daisys_tokens.json') as tokens_file:
             tokens = json.load(tokens_file)
             print('Loaded tokens from "daisys_tokens.json".')
             return tokens['access_token'], tokens['refresh_token']
     except (FileNotFoundError, json.JSONDecodeError):
         return None, None

 ACCESS_TOKEN, REFRESH_TOKEN = load_tokens()

 def store_tokens(access_token: str, refresh_token: str):
     """A function to store the access and refresh tokens to a local file.  In practice you
     might store this somewhere like in a database or larger configuration.

     """
     with open('daisys_tokens.json', 'w') as tokens_file:
         json.dump({'access_token': access_token,
                    'refresh_token': refresh_token},
                   tokens_file)
         print('Stored new tokens in "daisys_tokens.json".')

 def initial_login():
     """Initially retrieve access and refresh tokens through a normal login."""
     # Initial login is only required if we don't have an access token yet.
     print(f'Initial login, attempting to log in with {EMAIL} to retrieve an access token.')
     with DaisysAPI('speak', email=EMAIL, password=PASSWORD) as speak:
         # Say what should happen when tokens are retrieved or changed.
         speak.token_callback = store_tokens

         # Explicit login is only necessary if no other operations occur here, otherwise
         # login is automatic.
         speak.login()

         # Login enables auto-logout.  Disable it so that the token will not be invalidated.
         speak.auto_logout = False

         print('Run again to use stored access token!')

 def subsequent_login():
     """In subsequent uses, the previously stored access token can be used directly."""
     print('Using previously stored access token.')
     with DaisysAPI('speak', access_token=ACCESS_TOKEN, refresh_token=REFRESH_TOKEN) as speak:
         speak.token_callback = store_tokens

         # Here we are just showing how to authenticate with the Daisys API using an access
         # token instead of the password, so we just list some voices.  See hello_daisys.py
         # for an example of how to generate audio!

         # Get a list of all voices
         voices = speak.get_voices()
         print('Found voices:', [voice.name for voice in voices])

 def main():
     if ACCESS_TOKEN:
         subsequent_login()
     else:
         initial_login()

 if __name__=='__main__':
     try:
         main()
     except HTTPStatusError as e:
         try:
             print(f'HTTP error status {e.response.status_code}: {e.response.json()["detail"]}, {e.request.url}')
         except:
             print(f'HTTP error status {e.response.status_code}: {e.response.text}, {e.request.url}')

  Example: curl example

   This example shows:

    1. How to use curl and jq in a shell script to access the Daisys API.

    2. How to create the synchronous client using a context manager.

    3. Get a list of voices and select the last one.

    4. Reference the voice to generate audio (a “take”) for some text.

    5. Download the resulting audio.

    6. Play the audio using aplay (Linux).

   To run it, you must supply your email and password in the respecting
   environment variables, as shown below.

   Requires: curl, jq.

   Example output

 $ curl -O https://raw.githubusercontent.com/daisys-ai/daisys-api-python/main/examples/curl_example.sh
 $ jq --version  # "jq" is needed for the example program to parse API responses
 $ export DAISYS_EMAIL=user@example.com
 $ export DAISYS_PASSWORD=example_password123
 $ bash examples/curl_example.sh
 Found Daisys Speak API  {"version":1,"minor":0}
 GET https://api.daisys.ai/v1/speak/voices
 "Deirdre" is speaking!
 POST https://api.daisys.ai/v1/speak/takes/generate: {"voice_id": "v01hasgezqjcsnc91zdfzpx0apj",
 "text": "Hello there, I am Daisys!", "prosody": {"pace": -8, "pitch": 2, "expression": 8}}
 Take is "waiting".
 GET https://api.daisys.ai/v1/speak/takes/t01hawm80qzj60bf2w9z0np7wej
 Take is "started".
 GET https://api.daisys.ai/v1/speak/takes/t01hawm80qzj60bf2w9z0np7wej
 Take is "ready".
 Getting audio!
 GET https://api.daisys.ai/v1/speak/takes/t01hawm80qzj60bf2w9z0np7wej/wav
 Wrote 'hello_daisys.wav'.
 Playing WAVE 'hello_daisys.wav' : Signed 16 bit Little Endian, Rate 44100 Hz, Mono

   The “Playing” message will only appear if you have the aplay command
   installed, otherwise you may play the resulting hello_daisys.wav file in
   any audio player.

   examples/curl_example.sh

 #!/bin/bash

 # The following is an example of how to use the Daisys API for generating a voice and then
 # using it in a speech generation task using the "curl" program.  The API generates
 # "takes" representing one or more sentences from a speaker.

 # This program downloads the resulting .wav file and tries to play it using "aplay" if
 # that program is available.

 set -e  # Stop if we hit any problems along the way.

 EMAIL="${DAISYS_EMAIL:=user@example.com}"
 PASSWORD="${DAISYS_PASSWORD:=example_password}"

 DAISYS_AUTH="${DAISYS_AUTH_URL:=https://api.daisys.ai}"
 DAISYS="${DAISYS_API_URL:=https://api.daisys.ai}"
 API="$DAISYS/v1"
 SPEAK="$API/speak"

 TOKEN=$(curl -s -X POST -H 'Content-Type: application/json' -d '{"email": "'$EMAIL'", "password": "'$PASSWORD'"}' $DAISYS_AUTH/auth/login | jq -r ".access_token")

 AUTH="Authorization: Bearer $TOKEN"

 # Some functions for authenticated GET and POST methods using curl.
 speak_get() {
     echo "GET $SPEAK/$1" >/dev/stderr
     curl -s -L -H "$AUTH" "$SPEAK/$1"
 }
 speak_post() {
     echo "POST $SPEAK/$1: $2" >/dev/stderr
     curl -s -H "Content-Type: application/json" -H "$AUTH" -d "$2" "$SPEAK/$1"
 }

 VERSION=$(curl -s $API/speak/version)
 echo 'Found Daisys Speak API ' $VERSION

 # Get a list of all voices, select the last one.
 VOICE=$(speak_get voices | jq '.[-1]')
 if [ "$VOICE" = null ]; then
     echo No voices found.
     MODEL=$(speak_get models | jq '.[-1]')
     if [ "$MODEL" = null ]; then
         echo No models found.
         exit 1
     fi
     echo Using model $(echo $MODEL | jq .displayname)
     echo Generating a voice.
     VOICE=$(speak_post voices/generate '{"name": "Tina", "gender": "female", "model": '$(echo $MODEL | jq .name)'}')
 fi
 echo "$(echo $VOICE | jq .name) is speaking!"
 VOICE_ID="$(echo $VOICE | jq .voice_id)"

 TAKE=$(speak_post takes/generate '{"voice_id": '$VOICE_ID', "text": "Hello there, I am Daisys!", "prosody": {"pace": -8, "pitch": 2, "expression": 8}}')
 TAKE_ID="$(echo $TAKE | jq -r .take_id)"
 echo "Take is $(echo $TAKE | jq .status)."
 while [ $(echo $TAKE | jq -r .status) != 'ready' ] && [ $(echo $TAKE | jq -r .status) != 'error' ]; do
     sleep 0.5
     TAKE=$(speak_get takes/$TAKE_ID)
     echo "Take is $(echo $TAKE | jq .status)."
 done

 echo "Getting audio!"
 speak_get takes/$TAKE_ID/wav > hello_daisys.wav
 echo "Wrote 'hello_daisys.wav'."

 # Play the audio if we have aplay (Linux), otherwise just print a nice message.
 if which aplay >/dev/null; then
     aplay hello_daisys.wav
 else
     echo "aplay not found, but audio was written to 'hello_daisys.wav'."
 fi

Daisys API input

   The following examples can be used to see how the Daisys API client
   library for Python can be used.

   For generating speech audio, the Daisys API supports input text that can
   include certain directives.

  Input Text Customization

   Our models employ a powerful Normalizer (Advanced Text and SSML Tag
   Processing tool) designed to process and normalize text, making it more
   readable and coherent. It is equipped with a default pipeline of
   operations to apply on the input text, but it also allows for customizing
   the normalization process according to specific needs.

   The Normalizer applies a series of pre-defined steps by default to
   transform the text:

   Default pipeline:

    1. Abbreviations: Converts common abbreviations like “Mr.” to their full
       spoken versions (e.g., “Mister”).

    2. Acronyms: Acronyms will be processed and insert small pause between
       letter for better pronunciation (e.g., “10 AM” to “10 A M”).

    3. Road numbers: Converts road numbers by inserting a space, similar to
       acronyms (e.g. “A263” becomes “A 263”).

    4. URLs: Replaces URLs with a more human-readable description. (e.g. “As
       an example of https://google.com” to “As an example of google dot
       com.”)

    5. Numbers: Converts numerical expressions to their spoken form (e.g.,
       “10” to “ten”, “$40” to “forty dollars”).

    6. Punctuation Repeats: Simplifies repeated punctuation (e.g., “Hey!!!!”
       to “Hey!”).

    7. Units: Converts units such as “km/h” to their spoken form
       (“kilometer(s) per hour”).

  Advanced Capabilities:

   The Normalizer also provides advanced functionalities to handle SSML tags.
   It follows SSML is based on the World Wide Web Consortium’s “Speech
   Synthesis Markup Language Version 1.0”. Partially supports voice , phoneme
   and say-as tags.

    Voice Tag

   The voice tag is used when a specific text segment comes from a different
   language than the base language of that model. Our advanced automatic
   language prediction algorithm captures these parts and inserts a voice tag
   with the proper language attribute. This allows the model to apply
   required language change for that section. This tag can be manually added
   to input text. Normalization will be applied based on the defined language
   for voice tag section.

   Example usage:

 Input:
   The parking season ticket was valid <voice language="nl">t/m 09-01-2010</voice>.
 Normalizer output:
   The parking season ticket was valid tot en met negen januari tweeduizend tien.

    Phoneme Tag

   The phoneme tag provides control over the phonemization for the model. The
   model will use this pronunciation.

   Example usage:

 De <phoneme ph="ɣ ə k l øː r d ə">gekleurde</phoneme> vlag van een land.

     📌 Note: Phonemes need to be separated by a space. In case of multiple
     words, they should be separated by the @ symbol (e.g. De ernstige
     kapitein → d ə @ ɛ r n s t ə ɣ ə @ k ɑ p i t ɛɪ n)

    Say-as Tag

   The say-as tags allow users to interpret some specific types of text in a
   certain way.

   Supported attributes: 1. spell-out 2. year 3. date 4. time

   Example usage:

 Input:
   Mijn naam spel je als <say-as interpret-as="spell-out">Fred</say-as>.
   Het was <say-as interpret-as="year">1944</say-as>.
   Ik vertrek om <say-as interpret-as="time">13.10</say-as>.
   Ik ben geboren op <say-as interpret-as="date">11.4.1984</say-as>.

 Output:
   Mijn naam spel je als F r e d.
   Het was negentien vierenveertig.
   Ik vertrek om tien over één.
   Ik ben geboren op elf april negentien vierentachtig.

    w Tag

   The <w> tag allows the user to select the correct pronunciation for a word
   based on the part of speech and meaning. The part-of-speech tags from the
   [Penn
   Treebank](https://www.ling.upenn.edu/courses/Fall_2003/ling001/penn_treebank_pos.html)
   are used.

   Example options:

     * <w role="daisys:VB">read</w> : verb, present tense

     * <w role="daisys:VBD">read</w> : verb, past tense

     * <w role="daisys:NN">wind</w> : noun

     * <w role="daisys:JJ">live</w> : adjective

     * <w role="daisys:RB">live</w> : adverb

     * <w role="daisys:NN" sense="daisys:DEFAULT">bass</w> : default
       meaning/pronunciation (in the example: the music-related sense)

     * <w role="daisys:NN" sense="daisys:SENSE_1">bass</w> : first
       non-default meaning/pronunciation (in the example: the fish)

    Emphasis Tag

   The <emphasis> tag allows to select a word for emphasis. Intonation and
   duration behaviour of the model will be modified in the text between the
   tags.

   The strength of the emphasis can be modulated by the level attribute,
   which by default is moderate but can take the following values:

     * <emphasis level="moderate">some text</emphasis>: somewhat emphasize
       the text

     * <emphasis level="strong">some text</emphasis>: more strongly emphasize
       the text

     * <emphasis level="down">some text</emphasis>: emphasize the text by
       going down in pitch

     * <emphasis level="none">some text</emphasis>: avoid that the model
       automatically emphasizes this text

   Note that the model will often choose which parts of a sentence to
   emphasize depending on context if no hints are provided, so this level of
   control is critical if you want to avoid that the wrong word is selected
   or the ensure the right word is selected for emphasis.

   The pause attribute may also be added to insert a pause after the word
   which can enhance emphasis, it takes the same values as the strength
   attribute of the <break> tag:

     * <emphasis level="moderate" pause="weak">some text</emphasis>: somewhat
       emphasize the text, including a short pause

     * <emphasis level="strong" pause="weak">some text</emphasis>: strongly
       emphasize the text, including a medium pause

     * <emphasis pause="weak">some text</emphasis>: somewhat emphasize the
       text, including a long pause

    Break Tag

   The <break> tag inserts a pause at a given place in the text. The duration
   of the pause can be controlled by the strength attribute:

     * <break strength="weak"/>: a short pause, such as after a comma

     * <break strength="medium"/>: a medium-length pause, such as between
       sentences

     * <break strength="strong"/>: a longer pause, such as between paragraphs

Daisys API top-level object

   class  daisys.factory.DaisysAPI(product='speak', version='v1', email:  str
   |  None  =  None, password:  str  |  None  =  None, access_token:  str  | 
   None  =  None, refresh_token:  str  |  None  =  None, daisys_url:  str  = 
   'https://api.daisys.ai', auth_url:  str  =  'https://api.daisys.ai')

           Factory class to get a Daisys API client.

           This class is intended (but not required) to be used in a with or
           async with clause.

                __init__(product='speak', version='v1', email:  str  |  None 
                =  None, password:  str  |  None  =  None, access_token:  str
                |  None  =  None, refresh_token:  str  |  None  =  None,
                daisys_url:  str  =  'https://api.daisys.ai', auth_url:  str 
                =  'https://api.daisys.ai')

                        Initialize the factory object for a given product and
                        version.

                        This object is intended to be short lived, only used
                        to provide a client object.

                        Login or token details be be optionally provided. If
                        they are not provided here, they may be later
                        provided to the client by calling client.login().

                             Parameters:
                                        * product – The product to retrieve a
                                          client for.

                                        * version – The version of the
                                          product to retrieve a client for.

                                        * email – Optionally, email to use
                                          for logging in.

                                        * password – Optionally, password to
                                          use for logging in.

                                        * access_token – Optionally, access
                                          token to use. Specify if login was
                                          already performed.

                                        * refresh_token – Optionally, refresh
                                          token to use. Specify if login was
                                          already performed.

                                        * daisys_url – For overriding default
                                          API URL, usually not needed.

                                        * auth_url – For overriding default
                                          authentication URL, usually not
                                          needed.

                get_async_client() → DaisysAsyncSpeakClientV1

                        Retrieve a client for asynchronous usage of the
                        Daisys Speak API.

                get_client() → DaisysSyncSpeakClientV1

                        Retrieve a client for synchronous usage of the Daisys
                        Speak API.

Daisys API clients

   These objects should not be instantiated directly, but accessed through
   the DaisysAPI top-level factory object.

   The synchronous client makes requests using synchronous, blocking calls.
   The asynchronous client uses an asyncio event loop. You should choose
   whichever implementation is most convenient for your application.

   class  daisys.v1.speak.sync_client.DaisysSyncSpeakClientV1(auth_url:  str,
   product_url:  str, email:  str, password:  str, access_token:  str,
   refresh_token:  str)

           Wrapper for Daisys v1 API endpoints, synchronous version.

                close()

                        To be called when object is destroyed, to ensure any
                        open HTTP connections are cleanly closed. This is
                        done automatically if the client was created through
                        a context manager.

                delete_take(take_id:  str, raise_on_error:  bool  =  True) →
                bool

                        Delete a take. The take will no longer appear in
                        return values from get_takes.

                             Parameters:
                                        * take_id – the id of a take to
                                          delete.

                                        * raise_on_error – If True (default)
                                          a DaisysTakeDeletionException error
                                          will be raised if the take was not
                                          found. (That is, if the function
                                          would have returned False.)

                             Returns:

                                     True if the take was deleted
                                     successfully, otherwise False.

                             Return type:

                                     bool

                        Note that HTTP exceptions may be thrown for errors
                        other than a take not being found.

                delete_voice(voice_id:  str, raise_on_error:  bool  =  True)
                → bool

                        Delete a voice. The voice will no longer appear in
                        return values from get_voices.

                             Parameters:
                                        * voice_id – the id of a voice to
                                          delete.

                                        * raise_on_error – If True (default)
                                          a DaisysVoiceDeletionException
                                          error will be raised if the voice
                                          was not found. (That is, if the
                                          function would have returned
                                          False.)

                             Returns:

                                     True if the voice was deleted
                                     successfully, otherwise False.

                             Return type:

                                     bool

                        Note that HTTP exceptions may be thrown for errors
                        other than a voice not being found.

                generate_take(voice_id:  str, text:  str, override_language: 
                str  |  None  =  None, style:  list[str]  |  None  =  None,
                prosody:  SimpleProsody  |  AffectProsody  |  SignalProsody 
                |  None  =  None, status_webhook:  str  |  None  =  None,
                done_webhook:  str  |  None  =  None, wait:  bool  =  True,
                raise_on_error:  bool  =  True, timeout:  float  |  None  = 
                None) → TakeResponse

                        Generate a “take”, an audio file containing an
                        utterance of the given text by the given voice.

                             Parameters:
                                        * voice_id – The id of the voice to
                                          be used for generating audio. The
                                          voice is attached to a specific
                                          model.

                                        * text – The text that the voice
                                          should say.

                                        * override_language – Normally a
                                          language classifier is used to
                                          detect the language of the speech;
                                          this allows for multilingual
                                          sentences. However, if the language
                                          should be enforced, it should be
                                          provided here. Currently accepted
                                          values are “nl-NL” and “en-GB”.

                                        * style – A list of styles to enable
                                          when speaking. Note that most
                                          styles are mutually exclusive, so a
                                          list of 1 value should be provided.
                                          Accepted styles can be retrieved
                                          from the associated voice’s
                                          VoiceInfo.styles or the model’s
                                          TTSModel.styles field. Note that
                                          not all models support styles, thus
                                          this can be left empty if specific
                                          styles are not desired.

                                        * prosody – The characteristics of
                                          the desired speech not determined
                                          by the voice or style. Here you can
                                          provide a SimpleProsody or most
                                          models also accept the more
                                          detailed AffectProsody.

                                        * status_webhook – An optional URL to
                                          be called using POST whenever the
                                          take’s status changes, with
                                          TakeResponse in the body content.

                                        * done_webhook – An optional URL to
                                          be called exactly once using POST
                                          when the take is READY, ERROR, or
                                          TIMEOUT, with TakeResponse in the
                                          body content.

                                        * wait – if True, wait for take to be
                                          ready before returning.

                                        * raise_on_error – If True (default)
                                          a DaisysTakeGeenerateException
                                          error will be raised. If this
                                          behavior is not desired, set to
                                          False.

                                        * timeout – Time limit to wait, in
                                          seconds. Note that if timeout is
                                          specified, some results may not
                                          have a “done” status (ready or
                                          error).

                             Returns:

                                     Information about the take being
                                     generated, including status.

                             Return type:

                                     TakeResponse

                generate_takes(request:  list[TakeGenerate], wait:  bool  = 
                True, raise_on_error:  bool  =  True, timeout:  float  | 
                None  =  None) → list[TakeResponse]

                        Generate several “takes”, each corresponding to an
                        audio file containing an utterance of the given text
                        by the given voice.

                             Parameters:
                                        * request – a list of
                                          list[TakeGenerate] objects
                                          describing multiple take generation
                                          requests.

                                        * wait – if True, wait for all takes
                                          to be ready before returning.

                                        * raise_on_error – If True (default)
                                          a DaisysTakeGeenerateException
                                          error will be raised. If this
                                          behavior is not desired, set to
                                          False.

                                        * timeout – Time limit to wait, in
                                          seconds. Note that if timeout is
                                          specified, some results may not
                                          have a “done” status (ready or
                                          error).

                             Returns:

                                     a list of TakeResponse objects
                                     containing information on the generation
                                     status of each result.

                             Return type:

                                     list[TakeResponse]

                generate_voice(name:  str, model:  str, gender:  VoiceGender,
                description:  str  |  None  =  None, default_style: 
                list[str]  |  None  =  None, default_prosody:  SimpleProsody 
                |  AffectProsody  |  SignalProsody  |  None  =  None,
                example_take:  TakeGenerateWithoutVoice  |  None  =  None,
                done_webhook:  str  |  None  =  None, wait:  bool  =  True,
                raise_on_error:  bool  =  True, timeout:  float  |  None  = 
                None) → VoiceInfo

                        Generate a random, novel voice for a given model with
                        desired properties.

                             Parameters:
                                        * name – A name to give the voice,
                                          may be any string, and does not
                                          need to be unique.

                                        * model – The name of the model for
                                          this voice.

                                        * gender – The gender of this voice.

                                        * description – The description of
                                          this voice.

                                        * default_style – An optional list of
                                          styles to associate with this voice
                                          by default. It can be overriden by
                                          a take that uses this voice. Note
                                          that most styles are mutually
                                          exclusive, and not all models
                                          support styles.

                                        * default_prosody – An optional
                                          default prosody to associate with
                                          this voice. It can be overridden by
                                          a take that uses this voice.

                                        * example_take – Information on the
                                          take to generate as an example of
                                          this voice.

                                        * done_webhook – An optional URL to
                                          call exactly once using POST when
                                          the voice is available, with
                                          VoiceInfo in the body content.

                                        * wait – True to wait for the result,
                                          or False to continue without
                                          waiting.

                                        * raise_on_error – If True (default)
                                          a DaisysVoiceGenerateException
                                          error will be raised if an error
                                          status is detected in one of the
                                          takes. If this behavior is not
                                          desired, set to False.

                                        * timeout – Time limit to wait, in
                                          seconds. Note that if timeout is
                                          specified, some results may not
                                          have a “done” status (ready or
                                          error).

                             Returns:

                                     Information about the generated voice.

                             Return type:

                                     VoiceInfo

                get_model(model_name:  str) → TTSModel

                        Get information about a model.

                             Parameters:

                                     model_name – The name of the model,
                                     which is a unique identifier.

                             Returns:

                                     Information about the model.

                             Return type:

                                     TTSModel

                get_models() → list[TTSModel]

                        Get information about all available models.

                             Returns:

                                     Information about each model.

                             Return type:

                                     list[TTSModel]

                get_take(take_id:  str) → TakeResponse

                        Get information about a specific take.

                             Parameters:

                                     take_id – Unique identifier for a take.

                             Returns:

                                     Information about the requested take.

                             Return type:

                                     TakeResponse

                get_take_audio(take_id:  str, file:  str  |  None  =  None,
                format:  str  =  'wav') → bytes

                        Get audio associated with a take.

                             Parameters:
                                        * take_id – A take_id to retrieve the
                                          audio for.

                                        * file – Optionally, the filename of
                                          a file to write, or a file stream
                                          to write to.

                                        * format – A supported format, must
                                          be one of ‘wav’, ‘mp3’, ‘flac’,
                                          ‘m4a’. Note: only ‘wav’ may be
                                          retrieved without waiting for
                                          ‘ready’ status.

                             Returns:

                                     The content of the audio file associated
                                     with the requested take.

                             Return type:

                                     bytes

                get_take_audio_url(take_id:  str, format:  str  =  'wav') →
                str

                        Get the signed URL for audio associated with a take.
                        May be used to provide the URL to a download or
                        streaming client that does not have the API access
                        token.

                             Parameters:
                                        * take_id – A take_id to retrieve the
                                          audio URL for.

                                        * format – A supported format, msut
                                          be one of ‘wav’, ‘mp3’, ‘flac’,
                                          ‘m4a’. Note: only ‘wav’ may be
                                          retrieved without waiting for
                                          ‘ready’ status.

                             Returns:

                                          The URL that can be used to
                                          download the content of the

                                                  audio associated with the
                                                  requested take.

                             Return type:

                                     str

                get_takes(take_ids:  list[str]  |  None  =  None, length: 
                int  |  None  =  None, page:  int  |  None  =  None, older: 
                int  |  None  =  None, newer:  int  |  None  =  None) →
                list[TakeResponse]

                        Get a list of takes, optionally filtered.

                             Parameters:
                                        * take_ids – A list of specific takes
                                          to retrieve.

                                        * length – Maximum number of voices
                                          to return. Default: unlimited.

                                        * page – Return page “page” of length
                                          “length”. Default: 1.

                                        * older – Find voices older than or
                                          equal to this timestamp
                                          (milliseconds).

                                        * newer – Find voices newer than or
                                          equal to this timestamp
                                          (milliseconds).

                             Returns:

                                     Information about each take found. Empty
                                     list if none found.

                             Return type:

                                     list[TakeResponse]

                get_voice(voice_id:  str) → VoiceInfo

                        Get information about a voice.

                             Parameters:

                                     voice_id – The unique identififer for a
                                     voice.

                             Returns:

                                     Information about the voice.

                             Return type:

                                     VoiceInfo

                get_voices(length:  int  |  None  =  None, page:  int  | 
                None  =  None, older:  int  |  None  =  None, newer:  int  | 
                None  =  None) → list[VoiceInfo]

                        Get a list of voices, optionally filtered.

                             Parameters:
                                        * length – Maximum number of voices
                                          to return. Default: unlimited.

                                        * page – Return page “page” of length
                                          “length”. Default: 1.

                                        * older – Find voices older than or
                                          equal to this timestamp
                                          (milliseconds).

                                        * newer – Find voices newer than or
                                          equal to this timestamp
                                          (milliseconds).

                             Returns:

                                     Information about each voice found.

                             Return type:

                                     list[VoiceInfo]

                login(email:  str  |  None  =  None, password:  str  |  None 
                =  None) → bool

                        Log in to the Daisys API using the provided
                        credentials.

                        If successful, nothing is returned. An access token
                        is stored in the client for use in future requests.
                        May raise:

                             * DaisysCredentialsError: if insufficient
                               credentials are provided.

                             * httpx.HTTPStatusError(401): if credentials do
                               not successfully authenticate.

                             Parameters:
                                        * email – User name for the Daisys
                                          API credentials.

                                        * password – Password for the Daisys
                                          API credentials.

                login_refresh() → bool  |  None

                        Refresh access and refresh tokens for API
                        authorization. This function does not normally need
                        to be called explicitly, since the authorization
                        credentials shall be renewed automatically when
                        needed, however it is provided in case there is a
                        need to do so explicitly by the user.

                             * httpx.HTTPStatusError(401): if credentials do
                               not successfully authenticate.

                             Returns:

                                          True if successful, False if
                                          unsuccessful, and None if no
                                          refresh

                                                  token was available.

                             Return type:

                                     Optional[bool]

                logout(refresh_token:  str  |  None  =  None) → bool

                        Log out of the Daisys API. Revokes the refresh token
                        and and forgets the access and refresh tokens.

                             * httpx.HTTPStatusError(401): if credentials do
                               not successfully authenticate.

                        Note that further requests may auto-login again.

                             Returns:

                                     True if logout was successful, False if
                                     no tokens were provided to revoke.

                             Return type:

                                     bool

                stream_take_audio(take_id:  str)

                        Stream the audio by providing an iterator over chunks
                        of bytes.

                             Parameters:

                                     take_id – A take_id to retrieve the
                                     audio URL for.

                             Returns:

                                     use “for” to read chunks of bytes for
                                     this take.

                             Return type:

                                     iterator

                update_voice(voice_id:  str, name:  str  |  None  =  None,
                gender:  VoiceGender  |  None  =  None, description:  str  | 
                None  =  None, default_style:  list[str]  |  None  =  None,
                default_prosody:  SimpleProsody  |  AffectProsody  | 
                SignalProsody  |  None  =  None, raise_on_error:  bool  = 
                True, **_kwargs) → bool

                        Update a voice.

                             Parameters:
                                        * voice_id – the id of a voice to
                                          update.

                                        * name – A name to give the voice,
                                          may be any string, and does not
                                          need to be unique.

                                        * gender – The gender of this voice.

                                        * description – The description of
                                          this voice.

                                        * default_style – An optional list of
                                          styles to associate with this voice
                                          by default. It can be overriden by
                                          a take that uses this voice. Note
                                          that most styles are mutually
                                          exclusive, and not all models
                                          support styles.

                                        * default_prosody – An optional
                                          default prosody to associate with
                                          this voice. It can be overridden by
                                          a take that uses this voice.

                                        * raise_on_error – If True (default)
                                          a DaisysVoiceUpdateException error
                                          will be raised if the voice was not
                                          found. (That is, if the function
                                          would have returned False.)

                             Returns:

                                     True if the voice was updated
                                     successfully, otherwise False.

                             Return type:

                                     bool

                        Note that HTTP exceptions may be thrown for errors
                        other than a voice not being found.

                version() → Version

                        Get the version information for the API in use.

                             Returns:

                                     An object containing version
                                     information.

                             Return type:

                                     Version

                wait_for_takes(take_ids:  str  |  list[str],
                sleep_seconds=0.5, callback:  Callable[[TakeResponse  | 
                list[TakeResponse]],  None]  |  None  =  None,
                async_callback:  Callable[[TakeResponse  | 
                list[TakeResponse]],  Awaitable[None]]  |  None  =  None,
                raise_on_error:  bool  =  True, timeout:  float  |  None  = 
                None) → TakeResponse  |  list[TakeResponse]

                        Wait for a take or list of takes to be ready.

                             Parameters:
                                        * take_ids – Either a single take_id,
                                          or a list of take_id to wait for at
                                          the same time. In the latter case,
                                          the function will return when all
                                          take_id are done.

                                        * sleep_seconds – The number of
                                          seconds to wait while polling the
                                          take status.

                                        * callback – A synchronous function
                                          to call whenever the status of one
                                          of the takes changes. The argument
                                          it receives corresponds to a list
                                          of all takes requested. (A single
                                          take will also be embedded in a
                                          list.)

                                        * async_callback – An asynchronous
                                          function to call whenever the
                                          status of one of the takes changes.
                                          The argument it receives
                                          corresponds to a list of all takes
                                          requested.

                                        * raise_on_error – If True (default)
                                          a DaisysTakeGeenerateException
                                          error will be raised if an error
                                          status is detected in one of the
                                          takes. If this behavior is not
                                          desired, set to False.

                                        * timeout – Time limit to wait, in
                                          seconds. Note that if timeout is
                                          specified, some results may not
                                          have a “done” status (ready or
                                          error).

                wait_for_voices(voice_ids:  str  |  list[str], sleep_seconds:
                float  =  0.5, raise_on_error:  bool  =  True, timeout: 
                float  |  None  =  None) → VoiceInfo  |  list[VoiceInfo]

                        Wait for a voice or list of voices to be ready.

                             Parameters:
                                        * voice_ids – Either a single
                                          voice_id, or a list of voice_id to
                                          wait for at the same time. In the
                                          latter case, the function will
                                          return when all voice_id are done.

                                        * sleep_seconds – The number of
                                          seconds to wait while polling the
                                          voice status.

                                        * raise_on_error – If True (default)
                                          a DaisysVoiceGeenerateException
                                          error will be raised if an error
                                          status is detected in one of the
                                          voices. If this behavior is not
                                          desired, set to False.

                                        * timeout – Time limit to wait, in
                                          seconds. Note that if timeout is
                                          specified, some results may not
                                          have a “done” status (ready or
                                          error).

                websocket(model:  str  |  None  =  None, voice_id:  str  | 
                None  =  None) → DaisysSyncSpeakWebsocketV1

                        Get an interface to the websocket that manages the
                        connection, allows making voice generate and take
                        generate reqeusts, and handles streaming the
                        resulting audio.

                        This provided interface is intended to be used in a
                        with clause.

                             Parameters:
                                        * model – a websocket connection
                                          requires specifying a model or
                                          voice

                                        * voice_id – if model is not
                                          provided, voice_id must be provided

                             Returns:

                                     DaisysSyncSpeakWebsocketV1

                websocket_url(model:  str  |  None  =  None, voice_id:  str 
                |  None  =  None, raise_on_error:  bool  =  True) → str

                        Get a URL for connecting a websocket. Must specify
                        model or voice_id in order to indicate the principle
                        model to be used on this connection.

                             Parameters:
                                        * model – the model for which we want
                                          to retrieve a websocket URL.

                                        * voice_id – the id of a voice for
                                          which we want to retrieve a
                                          websocket URL.

                                        * raise_on_error – If True (default)
                                          an error will be raised if the
                                          voice was not found or the URL
                                          could not be retrieved.

                             Returns:

                                     The URL to connect a websocket to.

                             Return type:

                                     str

   class  daisys.v1.speak.async_client.DaisysAsyncSpeakClientV1(auth_url: 
   str, product_url:  str, email:  str, password:  str, access_token:  str,
   refresh_token:  str)

           Wrapper for Daisys v1 API endpoints, asynchronous version.

                async  close()

                        To be called when object is destroyed, to ensure any
                        open HTTP connections are cleanly closed. This is
                        done automatically if the client was created through
                        a context manager.

                async  delete_take(take_id:  str, raise_on_error:  bool  = 
                True) → bool

                        Delete a take. The take will no longer appear in
                        return values from get_takes.

                             Parameters:
                                        * take_id – the id of a take to
                                          delete.

                                        * raise_on_error – If True (default)
                                          a DaisysTakeDeletionException error
                                          will be raised if the take was not
                                          found. (That is, if the function
                                          would have returned False.)

                             Returns:

                                     True if the take was deleted
                                     successfully, otherwise False.

                             Return type:

                                     bool

                        Note that HTTP exceptions may be thrown for errors
                        other than a take not being found.

                async  delete_voice(voice_id:  str, raise_on_error:  bool  = 
                True) → bool

                        Delete a voice. The voice will no longer appear in
                        return values from get_voices.

                             Parameters:
                                        * voice_id – the id of a voice to
                                          delete.

                                        * raise_on_error – If True (default)
                                          a DaisysVoiceDeletionException
                                          error will be raised if the voice
                                          was not found. (That is, if the
                                          function would have returned
                                          False.)

                             Returns:

                                     True if the voice was deleted
                                     successfully, otherwise False.

                             Return type:

                                     bool

                        Note that HTTP exceptions may be thrown for errors
                        other than a voice not being found.

                async  generate_take(voice_id:  str, text:  str,
                override_language:  str  |  None  =  None, style:  list[str] 
                |  None  =  None, prosody:  SimpleProsody  |  AffectProsody 
                |  SignalProsody  |  None  =  None, status_webhook:  str  | 
                None  =  None, done_webhook:  str  |  None  =  None, wait: 
                bool  =  True, raise_on_error:  bool  =  True, timeout: 
                float  |  None  =  None) → TakeResponse

                        Generate a “take”, an audio file containing an
                        utterance of the given text by the given voice.

                             Parameters:
                                        * voice_id – The id of the voice to
                                          be used for generating audio. The
                                          voice is attached to a specific
                                          model.

                                        * text – The text that the voice
                                          should say.

                                        * override_language – Normally a
                                          language classifier is used to
                                          detect the language of the speech;
                                          this allows for multilingual
                                          sentences. However, if the language
                                          should be enforced, it should be
                                          provided here. Currently accepted
                                          values are “nl-NL” and “en-GB”.

                                        * style – A list of styles to enable
                                          when speaking. Note that most
                                          styles are mutually exclusive, so a
                                          list of 1 value should be provided.
                                          Accepted styles can be retrieved
                                          from the associated voice’s
                                          VoiceInfo.styles or the model’s
                                          TTSModel.styles field. Note that
                                          not all models support styles, thus
                                          this can be left empty if specific
                                          styles are not desired.

                                        * prosody – The characteristics of
                                          the desired speech not determined
                                          by the voice or style. Here you can
                                          provide a SimpleProsody or most
                                          models also accept the more
                                          detailed AffectProsody.

                                        * status_webhook – An optional URL to
                                          be called using POST whenever the
                                          take’s status changes, with
                                          TakeResponse in the body content.

                                        * done_webhook – An optional URL to
                                          be called exactly once using POST
                                          when the take is READY, ERROR, or
                                          TIMEOUT, with TakeResponse in the
                                          body content.

                                        * wait – if True, wait for take to be
                                          ready before returning.

                                        * raise_on_error – If True (default)
                                          a DaisysTakeGeenerateException
                                          error will be raised. If this
                                          behavior is not desired, set to
                                          False.

                                        * timeout – Time limit to wait, in
                                          seconds. Note that if timeout is
                                          specified, some results may not
                                          have a “done” status (ready or
                                          error).

                             Returns:

                                     Information about the take being
                                     generated, including status.

                             Return type:

                                     TakeResponse

                async  generate_takes(request:  list[TakeGenerate], wait: 
                bool  =  True, raise_on_error:  bool  =  True, timeout: 
                float  |  None  =  None) → list[TakeResponse]

                        Generate several “takes”, each corresponding to an
                        audio file containing an utterance of the given text
                        by the given voice.

                             Parameters:
                                        * request – a list of
                                          list[TakeGenerate] objects
                                          describing multiple take generation
                                          requests.

                                        * wait – if True, wait for all takes
                                          to be ready before returning.

                                        * raise_on_error – If True (default)
                                          a DaisysTakeGeenerateException
                                          error will be raised. If this
                                          behavior is not desired, set to
                                          False.

                                        * timeout – Time limit to wait, in
                                          seconds. Note that if timeout is
                                          specified, some results may not
                                          have a “done” status (ready or
                                          error).

                             Returns:

                                     a list of TakeResponse objects
                                     containing information on the generation
                                     status of each result.

                             Return type:

                                     list[TakeResponse]

                async  generate_voice(name:  str, model:  str, gender: 
                VoiceGender, description:  str  |  None  =  None,
                default_style:  list[str]  |  None  =  None, default_prosody:
                SimpleProsody  |  AffectProsody  |  SignalProsody  |  None  =
                None, example_take:  TakeGenerateWithoutVoice  |  None  = 
                None, done_webhook:  str  |  None  =  None, wait:  bool  | 
                None  =  True, raise_on_error:  bool  |  None  =  True,
                timeout:  float  |  None  =  None) → VoiceInfo

                        Generate a random, novel voice for a given model with
                        desired properties.

                             Parameters:
                                        * name – A name to give the voice,
                                          may be any string, and does not
                                          need to be unique.

                                        * model – The name of the model for
                                          this voice.

                                        * gender – The gender of this voice.

                                        * description – The description of
                                          this voice.

                                        * default_style – An optional list of
                                          styles to associate with this voice
                                          by default. It can be overriden by
                                          a take that uses this voice. Note
                                          that most styles are mutually
                                          exclusive, and not all models
                                          support styles.

                                        * default_prosody – An optional
                                          default prosody to associate with
                                          this voice. It can be overridden by
                                          a take that uses this voice.

                                        * example_take – Information on the
                                          take to generate as an example of
                                          this voice.

                                        * done_webhook – An optional URL to
                                          call exactly once using POST when
                                          the voice is available, with
                                          VoiceInfo in the body content.

                                        * wait – True to wait for the result,
                                          or False to continue without
                                          waiting.

                                        * raise_on_error – If True (default)
                                          a DaisysVoiceGenerateException
                                          error will be raised if an error
                                          status is detected in one of the
                                          takes. If this behavior is not
                                          desired, set to False.

                                        * timeout – Time limit to wait, in
                                          seconds. Note that if timeout is
                                          specified, some results may not
                                          have a “done” status (ready or
                                          error).

                             Returns:

                                     Information about the generated voice.

                             Return type:

                                     VoiceInfo

                async  get_model(model_name:  str) → TTSModel

                        Get information about a model.

                             Parameters:

                                     model_name – The name of the model,
                                     which is a unique identifier.

                             Returns:

                                     Information about the model.

                             Return type:

                                     TTSModel

                async  get_models() → list[TTSModel]

                        Get information about all available models.

                             Returns:

                                     Information about each model.

                             Return type:

                                     list[TTSModel]

                async  get_take(take_id:  str) → TakeResponse

                        Get information about a specific take.

                             Parameters:

                                     take_id – Unique identifier for a take.

                             Returns:

                                     Information about the requested take.

                             Return type:

                                     TakeResponse

                async  get_take_audio(take_id:  str, file:  str  |  None  = 
                None, format:  str  =  'wav') → bytes

                        Get audio associated with a take.

                             Parameters:
                                        * take_id – A take_id to retrieve the
                                          audio for.

                                        * file – Optionally, the filename of
                                          a file to write, or a file stream
                                          to write to.

                                        * format – A supported format, must
                                          be one of ‘wav’, ‘mp3’, ‘flac’,
                                          ‘m4a’. Note: only ‘wav’ may be
                                          retrieved without waiting for
                                          ‘ready’ status.

                             Returns:

                                     The content of the audio file associated
                                     with the requested take.

                             Return type:

                                     bytes

                async  get_take_audio_url(take_id:  str, format:  str  = 
                'wav') → str

                        Get the signed URL for audio associated with a take.
                        May be used to provide the URL to a download or
                        streaming client that does not have the API access
                        token.

                             Parameters:
                                        * take_id – A take_id to retrieve the
                                          audio URL for.

                                        * format – A supported format, msut
                                          be one of ‘wav’, ‘mp3’, ‘flac’,
                                          ‘m4a’. Note: only ‘wav’ may be
                                          retrieved without waiting for
                                          ‘ready’ status.

                             Returns:

                                          The URL that can be used to
                                          download the content of the

                                                  audio associated with the
                                                  requested take.

                             Return type:

                                     str

                async  get_takes(take_ids:  list[str]  |  None  =  None,
                length:  int  |  None  =  None, page:  int  |  None  =  None,
                older:  int  |  None  =  None, newer:  int  |  None  =  None)
                → list[TakeResponse]

                        Get a list of takes, optionally filtered.

                             Parameters:
                                        * take_ids – A list of specific takes
                                          to retrieve.

                                        * length – Maximum number of voices
                                          to return. Default: unlimited.

                                        * page – Return page “page” of length
                                          “length”. Default: 1.

                                        * older – Find voices older than or
                                          equal to this timestamp
                                          (milliseconds).

                                        * newer – Find voices newer than or
                                          equal to this timestamp
                                          (milliseconds).

                             Returns:

                                     Information about each take found. Empty
                                     list if none found.

                             Return type:

                                     list[TakeResponse]

                async  get_voice(voice_id:  str) → VoiceInfo

                        Get information about a voice.

                             Parameters:

                                     voice_id – The unique identififer for a
                                     voice.

                             Returns:

                                     Information about the voice.

                             Return type:

                                     VoiceInfo

                async  get_voices(length:  int  |  None  =  None, page:  int 
                |  None  =  None, older:  int  |  None  =  None, newer:  int 
                |  None  =  None) → list[VoiceInfo]

                        Get a list of voices, optionally filtered.

                             Parameters:
                                        * length – Maximum number of voices
                                          to return. Default: unlimited.

                                        * page – Return page “page” of length
                                          “length”. Default: 1.

                                        * older – Find voices older than or
                                          equal to this timestamp
                                          (milliseconds).

                                        * newer – Find voices newer than or
                                          equal to this timestamp
                                          (milliseconds).

                             Returns:

                                     Information about each voice found.

                             Return type:

                                     list[VoiceInfo]

                async  login(email:  str  |  None  =  None, password:  str  |
                None  =  None) → bool

                        Log in to the Daisys API using the provided
                        credentials.

                        If successful, nothing is returned. An access token
                        is stored in the client for use in future requests.
                        May raise:

                             * DaisysCredentialsError: if insufficient
                               credentials are provided.

                             * httpx.HTTPCStatusError(401): if credentials do
                               not successfully authenticate.

                             Parameters:
                                        * email – User name for the Daisys
                                          API credentials.

                                        * password – Password for the Daisys
                                          API credentials.

                async  login_refresh() → bool  |  None

                        Refresh access and refresh tokens for API
                        authorization. This function does not normally need
                        to be called explicitly, since the authorization
                        credentials shall be renewed automatically when
                        needed, however it is provided in case there is a
                        need to do so explicitly by the user.

                             * httpx.HTTPStatusError(401): if credentials do
                               not successfully authenticate.

                             Returns:

                                          True if successful, False if
                                          unsuccessful, and None if no
                                          refresh

                                                  token was available.

                             Return type:

                                     Optional[bool]

                async  logout(refresh_token:  str  |  None  =  None) → bool

                        Log out of the Daisys API. Revokes the refresh token
                        and and forgets the access and refresh tokens.

                             * httpx.HTTPStatusError(401): if credentials do
                               not successfully authenticate.

                        Note that further requests may auto-login again.

                             Returns:

                                     True if logout was successful, False if
                                     no tokens were provided to revoke.

                             Return type:

                                     bool

                stream_take_audio(take_id:  str)

                        Stream the audio by providing an iterator over chunks
                        of bytes.

                             Parameters:

                                     take_id – A take_id to retrieve the
                                     audio URL for.

                             Returns:

                                     use “for” to read chunks of bytes for
                                     this take.

                             Return type:

                                     iterator

                async  update_voice(voice_id:  str, name:  str  |  None  = 
                None, gender:  VoiceGender  |  None  =  None, description: 
                str  |  None  =  None, default_style:  list[str]  |  None  = 
                None, default_prosody:  SimpleProsody  |  AffectProsody  | 
                SignalProsody  |  None  =  None, raise_on_error:  bool  = 
                True, **_kwargs) → bool

                        Update a voice. The voice will no longer appear in
                        return values from get_voices.

                             Parameters:
                                        * voice_id – the id of a voice to
                                          update.

                                        * name – A name to give the voice,
                                          may be any string, and does not
                                          need to be unique.

                                        * gender – The gender of this voice.

                                        * description – The description of
                                          this voice.

                                        * default_style – An optional list of
                                          styles to associate with this voice
                                          by default. It can be overriden by
                                          a take that uses this voice. Note
                                          that most styles are mutually
                                          exclusive, and not all models
                                          support styles.

                                        * default_prosody – An optional
                                          default prosody to associate with
                                          this voice. It can be overridden by
                                          a take that uses this voice.

                                        * raise_on_error – If True (default)
                                          a DaisysVoiceUpdateException error
                                          will be raised if the voice was not
                                          found. (That is, if the function
                                          would have returned False.)

                             Returns:

                                     True if the voice was updated
                                     successfully, otherwise False.

                             Return type:

                                     bool

                        Note that HTTP exceptions may be thrown for errors
                        other than a voice not being found.

                async  version() → Version

                        Get the version information for the API in use.

                             Returns:

                                     An object containing version
                                     information.

                             Return type:

                                     Version

                async  wait_for_takes(take_ids:  str  |  list[str],
                sleep_seconds=0.5, callback:  Callable[[TakeResponse  | 
                list[TakeResponse]],  None]  |  None  =  None,
                async_callback:  Callable[[TakeResponse  | 
                list[TakeResponse]],  Awaitable[None]]  |  None  =  None,
                raise_on_error:  bool  =  True, timeout:  float  |  None  = 
                None) → TakeResponse  |  list[TakeResponse]

                        Wait for a take or list of takes to be ready.

                             Parameters:
                                        * take_ids – Either a single take_id,
                                          or a list of take_id to wait for at
                                          the same time. In the latter case,
                                          the function will return when all
                                          take_id are done.

                                        * sleep_seconds – The number of
                                          seconds to wait while polling the
                                          take status.

                                        * callback – A synchronous function
                                          to call whenever the status of one
                                          of the takes changes. The argument
                                          it receives corresponds to a list
                                          of all takes requested. (A single
                                          take will also be embedded in a
                                          list.)

                                        * async_callback – An asynchronous
                                          function to call whenever the
                                          status of one of the takes changes.
                                          The argument it receives
                                          corresponds to a list of all takes
                                          requested.

                                        * raise_on_error – If True (default)
                                          a DaisysTakeGeenerateException
                                          error will be raised if an error
                                          status is detected in one of the
                                          takes. If this behavior is not
                                          desired, set to False.

                                        * timeout – Time limit to wait, in
                                          seconds. Note that if timeout is
                                          specified, some results may not
                                          have a “done” status (ready or
                                          error).

                async  wait_for_voices(voice_ids:  str  |  list[str],
                sleep_seconds:  float  =  0.5, raise_on_error:  bool  = 
                True, timeout:  float  |  None  =  None) → VoiceInfo  | 
                list[VoiceInfo]

                        Wait for a voice or list of voices to be ready.

                             Parameters:
                                        * voice_ids – Either a single
                                          voice_id, or a list of voice_id to
                                          wait for at the same time. In the
                                          latter case, the function will
                                          return when all voice_id are done.

                                        * sleep_seconds – The number of
                                          seconds to wait while polling the
                                          voice status.

                                        * raise_on_error – If True (default)
                                          a DaisysVoiceGeenerateException
                                          error will be raised if an error
                                          status is detected in one of the
                                          voices. If this behavior is not
                                          desired, set to False.

                                        * timeout – Time limit to wait, in
                                          seconds. Note that if timeout is
                                          specified, some results may not
                                          have a “done” status (ready or
                                          error).

                websocket(model:  str  |  None  =  None, voice_id:  str  | 
                None  =  None) → DaisysAsyncSpeakWebsocketV1

                        Get an interface to the websocket that manages the
                        connection, allows making voice generate and take
                        generate reqeusts, and handles streaming the
                        resulting audio.

                        This provided interface is intended to be used in an
                        async with clause.

                             Parameters:
                                        * model – a websocket connection
                                          requires specifying a model or
                                          voice

                                        * voice_id – if model is not
                                          provided, voice_id must be provided

                             Returns:

                                     DaisysAsyncSpeakWebsocketV1

                async  websocket_url(model:  str  |  None  =  None, voice_id:
                str  |  None  =  None, raise_on_error:  bool  =  True) → str

                        Get a URL for connecting a websocket. Must specify
                        model or voice_id in order to indicate the principle
                        model to be used on this connection.

                             Parameters:
                                        * model – the model for which we want
                                          to retrieve a websocket URL.

                                        * voice_id – the id of a voice for
                                          which we want to retrieve a
                                          websocket URL.

                                        * raise_on_error – If True (default)
                                          an error will be raised if the
                                          voice was not found or the URL
                                          could not be retrieved.

                             Returns:

                                     The URL to connect a websocket to.

                             Return type:

                                     str

Daisys API JSON models

   Pydantic classes representing the JSON interface for the Daisys API.

   class  daisys.v1.speak.models.AffectProsody(*, pitch:  int, pace:  int,
   valence:  int, dominance:  int, arousal:  int)

           Prosody features based on analysis of affect. See also parent
           class ProsodyFeatures for other fields.

                valence

                        The valence; -10 for negativity, 10 for positivity, 0
                        for neutral.

                             Type:

                                     int

                arousal

                        The arousal; -10 for unexcited, 10 for very excited,
                        0 for neutral.

                             Type:

                                     int

                dominance

                        The dominance; -10 for docile, 10 for commanding, 0
                        for neutral.

                             Type:

                                     int

                model_config:  ClassVar[ConfigDict]  =  {}

                        Configuration for the model, should be a dictionary
                        conforming to
                        [ConfigDict][pydantic.config.ConfigDict].

   class  daisys.v1.speak.models.ProsodyFeatures(*, pitch:  int, pace:  int)

           Base prosody features supported by all models.

                pitch

                        The normalized pitch; -10 to 10, where 0 is a neutral
                        pitch.

                             Type:

                                     int

                pace

                        The normalized pace; -10 to 10, where 0 is a neutral
                        pace.

                             Type:

                                     int

                model_config:  ClassVar[ConfigDict]  =  {}

                        Configuration for the model, should be a dictionary
                        conforming to
                        [ConfigDict][pydantic.config.ConfigDict].

   daisys.v1.speak.models.ProsodyFeaturesUnion

           A union type representing different prosody feature variations.

           alias of SimpleProsody | AffectProsody | SignalProsody

   class  daisys.v1.speak.models.ProsodyType(value)

           An enum representing different prosody feature types.

           Not all models accept all prosody types. See the prosody_types
           field of TTSModel.

                SIMPLE

                        corresponds with SimpleProsody

                AFFECT

                        corresponds with AffectProsody

                SIGNAL

                        corresponds with SignalProsody

                static  from_class(prosody:  SimpleProsody  |  AffectProsody 
                |  SignalProsody)

                        Return an enum value based on the prosody class
                        provided.

                             Parameters:

                                     prosody – The prosody object from which
                                     to derive the enum value.

                prosody(**kwargs)

                        Return a prosody object corresponding to this value,
                        initialized with the given arguments.

   class  daisys.v1.speak.models.SignalProsody(*, pitch:  int, pace:  int,
   tilt:  int, pitch_range:  int)

           Prosody features based on signal analysis. See also parent class
           ProsodyFeatures for other fields.

                tilt

                        The normalized spectral tilt; -10 for flat, 10 for
                        bright, 0 for neutral.

                             Type:

                                     int

                pitch_range

                        The normalized pitch range; -10 for flat, 10 for
                        highly varied pitch, 0 for neutral.

                             Type:

                                     int

                model_config:  ClassVar[ConfigDict]  =  {}

                        Configuration for the model, should be a dictionary
                        conforming to
                        [ConfigDict][pydantic.config.ConfigDict].

   class  daisys.v1.speak.models.SimpleProsody(*, pitch:  int, pace:  int,
   expression:  int)

           Simplified prosody features, supported by all models. See also
           parent class ProsodyFeatures for other fields.

                expression

                        The normalized “expression”; -10 to 10, where 0 is
                        neutral.

                             Type:

                                     int

                model_config:  ClassVar[ConfigDict]  =  {}

                        Configuration for the model, should be a dictionary
                        conforming to
                        [ConfigDict][pydantic.config.ConfigDict].

   class  daisys.v1.speak.models.Status(value)

           Represents the status of a take or voice generation process.

                WAITING

                        Item is waiting to be processed.

                STARTED

                        Processing has started for this item.

                PROGRESS_25

                        Item has been 25% processed.

                PROGRESS_50

                        Item has been 50% procesesd.

                PROGRESS_75

                        Item has been 75% procesesd.

                READY

                        Item is ready to be used; for takes, audio is
                        available.

                ERROR

                        An error occurred during processing of this item.

                TIMEOUT

                        Processing did not finish for this item.

           Note that TIMEOUT is used for very long intervals; it does not
           indicate a few seconds or minutes, but rather that an item has
           been in the queue for more than a day and has therefore been
           removed. It should only be considered to represent circumstances
           where processing errors were not detected by normal means.

   class  daisys.v1.speak.models.StreamMode(value)

           Whether a websocket messages should contain a whole part or chunks
           of parts.

           Note: upper case in Python, lower case in JSON.

                Values:

                        PARTS, CHUNKS

   class  daisys.v1.speak.models.StreamOptions(*, mode:  StreamMode  = 
   StreamMode.PARTS)

           Options for streaming.

                mode

                        The streaming mode to use.

                             Type:

                                     daisys.v1.speak.models.StreamMode

                model_config:  ClassVar[ConfigDict]  =  {}

                        Configuration for the model, should be a dictionary
                        conforming to
                        [ConfigDict][pydantic.config.ConfigDict].

   class  daisys.v1.speak.models.TTSModel(*, name:  str, displayname:  str,
   flags:  list[str]  =  [], languages:  list[str], genders: 
   list[VoiceGender], styles:  list[list[str]]  =  [], prosody_types: 
   list[ProsodyType], voice_inputs:  list[VoiceInputType]  |  None)

           Information about a speech model.

                name

                        The unique identifier of this model.

                             Type:

                                     str

                displayname

                        A friendlier name that might contain spaces.

                             Type:

                                     str

                flags

                        A list of flags that indicate some features of this
                        model.

                             Type:

                                     list[str]

                languages

                        A list of languages supported by this model.

                             Type:

                                     list[str]

                genders

                        A list of genders supported by this model.

                             Type:

                                     list[daisys.v1.speak.models.VoiceGender]

                styles

                        A list of style sets; each sublist is a list of
                        mutually exlusive style tags.

                             Type:

                                     list[list[str]]

                prosody_types

                        A list of which prosody types are supported by this
                        model.

                             Type:

                                     list[daisys.v1.speak.models.ProsodyType]

                voice_inputs

                        A list of which voice input types are supported by
                        this model.

                             Type:

                                     list[daisys.v1.speak.models.VoiceInputType]
                                     | None

                model_config:  ClassVar[ConfigDict]  =  {}

                        Configuration for the model, should be a dictionary
                        conforming to
                        [ConfigDict][pydantic.config.ConfigDict].

   class  daisys.v1.speak.models.TakeGenerate(*, text:  str,
   override_language:  str  |  None  =  None, style:  list[str]  |  None  = 
   None, prosody:  SimpleProsody  |  AffectProsody  |  SignalProsody  |  None
   =  None, status_webhook:  Webhook  |  None  =  None, done_webhook: 
   Webhook  |  None  =  None, voice_id:  str)

           Parameters necessary to generate a “take”, an audio file
           containing an utterance of the given text by the given voice. See
           TakeGenerateWithoutVoice for documentation on the remaining
           fields.

                voice_id

                        The id of the voice to be used for generating audio.
                        The voice is attached to a specific model.

                             Type:

                                     str

                model_config:  ClassVar[ConfigDict]  =  {}

                        Configuration for the model, should be a dictionary
                        conforming to
                        [ConfigDict][pydantic.config.ConfigDict].

   class  daisys.v1.speak.models.TakeGenerateWithoutVoice(*, text:  str,
   override_language:  str  |  None  =  None, style:  list[str]  |  None  = 
   None, prosody:  SimpleProsody  |  AffectProsody  |  SignalProsody  |  None
   =  None, status_webhook:  Webhook  |  None  =  None, done_webhook: 
   Webhook  |  None  =  None)

           Parameters necessary to generate a “take”, an audio file
           containing an utterance of the given text. No voice is provided
           here, for the purpose of embedding in VoiceGenerate for the voice
           example.

                text

                        The text that the voice should say.

                             Type:

                                     str

                override_language

                        Normally a language classifier is used to detect the
                        language of the speech; this allows for multilingual
                        sentences. However, if the language should be
                        enforced, it should be provided here. Currently
                        accepted values are “nl-NL” and “en-GB”.

                             Type:

                                     str | None

                style

                        A list of styles to enable when speaking. Note that
                        most styles are mutually exclusive, so a list of 1
                        value should be provided. Accepted styles can be
                        retrieved from the associated voice’s
                        VoiceInfo.styles or the model’s TTSModel.styles
                        field. Note that not all models support styles, thus
                        this can be left empty if specific styles are not
                        desired.

                             Type:

                                     list[str] | None

                prosody

                        The characteristics of the desired speech not
                        determined by the voice or style. Here you can
                        provide a SimpleProsody or most models also accept
                        the more detailed AffectProsody.

                             Type:

                                     daisys.v1.speak.models.SimpleProsody |
                                     daisys.v1.speak.models.AffectProsody |
                                     daisys.v1.speak.models.SignalProsody |
                                     None

                status_webhook

                        An optional URL to be called using POST whenever the
                        take’s status changes, with TakeResponse in the body
                        content.

                             Type:

                                     daisys.v1.speak.models.Webhook | None

                done_webhook

                        An optional URL to be called exactly once using POST
                        when the take is READY, ERROR, or TIMEOUT, with
                        TakeResponse in the body content.

                             Type:

                                     daisys.v1.speak.models.Webhook | None

                model_config:  ClassVar[ConfigDict]  =  {}

                        Configuration for the model, should be a dictionary
                        conforming to
                        [ConfigDict][pydantic.config.ConfigDict].

   class  daisys.v1.speak.models.TakeInfo(*, duration:  int, audio_rate: 
   int, normalized_text:  list[str])

           Some information available when a take is READY, attached to the
           TakeResponse.

                duration

                        The length of the audio in samples. To get the length
                        in seconds, divide by audio_rate.

                             Type:

                                     int

                audio_rate

                        The number of samples per second in the audio.

                             Type:

                                     int

                normalized_text

                        The text used for text-to-speech after normalization,
                        ie. translated from “as written” to “as spoken”.
                        Provided as a list of sentences.

                             Type:

                                     list[str]

                model_config:  ClassVar[ConfigDict]  =  {}

                        Configuration for the model, should be a dictionary
                        conforming to
                        [ConfigDict][pydantic.config.ConfigDict].

   class  daisys.v1.speak.models.TakeResponse(*, text:  str,
   override_language:  str  |  None  =  None, style:  list[str]  |  None  = 
   None, prosody:  SimpleProsody  |  AffectProsody  |  SignalProsody  |  None
   =  None, status_webhook:  Webhook  |  None  =  None, done_webhook: 
   Webhook  |  None  =  None, voice_id:  str, take_id:  str, status:  Status,
   timestamp_ms:  int, info:  TakeInfo  |  None  =  None)

           Information about a take, returned during and after take
           generation. Also includes fields from TakeGenerate.

                take_id

                        The unique identifier of this take.

                             Type:

                                     str

                status

                        The status of this take, whether it is ready, in
                        error, or in progress.

                             Type:

                                     daisys.v1.speak.models.Status

                timestamp_ms

                        The timestamp that this take generation was
                        requested, in milliseconds since epoch.

                             Type:

                                     int

                info

                        Information available when the take is READY, see
                        TakeInfo.

                             Type:

                                     daisys.v1.speak.models.TakeInfo | None

                model_config:  ClassVar[ConfigDict]  =  {}

                        Configuration for the model, should be a dictionary
                        conforming to
                        [ConfigDict][pydantic.config.ConfigDict].

   class  daisys.v1.speak.models.Version(*, version:  int, minor:  int)

           Represents the version of the API.

                version

                        The major version number of the API.

                             Type:

                                     int

                minor

                        The minor version number of the API.

                             Type:

                                     int

                model_config:  ClassVar[ConfigDict]  =  {}

                        Configuration for the model, should be a dictionary
                        conforming to
                        [ConfigDict][pydantic.config.ConfigDict].

   class  daisys.v1.speak.models.VoiceGender(value)

           Represents the gender of a voice.

           Note: upper case in Python, lower case in JSON.

                Values:

                        MALE, FEMALE, NONBINARY

   class  daisys.v1.speak.models.VoiceGenerate(*, name:  str, model:  str,
   gender:  VoiceGender, description:  str  |  None  =  None, default_style: 
   list[str]  |  None  =  None, default_prosody:  SimpleProsody  | 
   AffectProsody  |  SignalProsody  |  None  =  None, example_take: 
   TakeGenerateWithoutVoice  |  None  =  None, done_webhook:  Webhook  | 
   None  =  None)

           Parameters necessary to generate a voice.

                name

                        A name to give the voice, may be any string, and does
                        not need to be unique.

                             Type:

                                     str

                model

                        The name of the model for this voice. Refers to the
                        name entry in TTSModel.

                             Type:

                                     str

                gender

                        The gender of this voice.

                             Type:

                                     daisys.v1.speak.models.VoiceGender

                description

                        A description of this voice.

                             Type:

                                     str | None

                default_style

                        An optional list of styles to associate with this
                        voice by default. It can be overriden by a take that
                        uses this voice. Note that most styles are mutually
                        exclusive, and not all models support styles.

                             Type:

                                     list[str] | None

                default_prosody

                        An optional default prosody to associate with this
                        voice. It can be overridden by a take that uses this
                        voice.

                             Type:

                                     daisys.v1.speak.models.SimpleProsody |
                                     daisys.v1.speak.models.AffectProsody |
                                     daisys.v1.speak.models.SignalProsody |
                                     None

                example_take

                        Parameters for an example take to generate for this
                        voice. If not provided, a default example text will
                        be used, depending on the language of the model.

                             Type:

                                     daisys.v1.speak.models.TakeGenerateWithoutVoice
                                     | None

                done_webhook

                        An optional URL to call using POST when the voice is
                        available, with the response of VoiceInfo in the body
                        content. This shall be called once, after the voice
                        and example take have been generated.

                             Type:

                                     daisys.v1.speak.models.Webhook | None

                model_config:  ClassVar[ConfigDict]  =  {}

                        Configuration for the model, should be a dictionary
                        conforming to
                        [ConfigDict][pydantic.config.ConfigDict].

   class  daisys.v1.speak.models.VoiceInfo(*, name:  str, model:  str,
   gender:  VoiceGender, description:  str  |  None  =  None, default_style: 
   list[str]  |  None  =  None, default_prosody:  SimpleProsody  | 
   AffectProsody  |  SignalProsody  |  None  =  None, example_take: 
   TakeGenerateWithoutVoice  |  None  =  None, done_webhook:  Webhook  | 
   None  =  None, voice_id:  str, status:  Status, timestamp_ms:  int,
   example_take_id:  str  |  None  =  None)

           Information about a voice.

                voice_id

                        The unique identifier of this voice.

                             Type:

                                     str

                status

                        The status of this voice, whether it is ready, in
                        error, or in progress.

                             Type:

                                     daisys.v1.speak.models.Status

                timestamp_ms

                        The timestamp that this voice generation was
                        requested, in milliseconds since epoch.

                             Type:

                                     int

                example_take_id

                        An optional identifier for a take that represents an
                        example of this voice.

                             Type:

                                     str | None

                model_config:  ClassVar[ConfigDict]  =  {}

                        Configuration for the model, should be a dictionary
                        conforming to
                        [ConfigDict][pydantic.config.ConfigDict].

   class  daisys.v1.speak.models.VoiceUpdate(*, name:  str  |  None  =  None,
   gender:  VoiceGender  |  None  =  None, default_style:  list[str]  |  None
   =  None, default_prosody:  SimpleProsody  |  AffectProsody  | 
   SignalProsody  |  None  =  None)

           Update parameters of a voice.

                name

                        A name to give the voice, may be any string, and does
                        not need to be unique.

                             Type:

                                     str | None

                gender

                        The gender of this voice.

                             Type:

                                     daisys.v1.speak.models.VoiceGender |
                                     None

                default_style

                        An optional list of styles to associate with this
                        voice by default. It can be overriden by a take that
                        uses this voice. Note that most styles are mutually
                        exclusive, and not all models support styles.

                             Type:

                                     list[str] | None

                default_prosody

                        An optional default prosody to associate with this
                        voice. It can be overridden by a take that uses this
                        voice.

                             Type:

                                     daisys.v1.speak.models.SimpleProsody |
                                     daisys.v1.speak.models.AffectProsody |
                                     daisys.v1.speak.models.SignalProsody |
                                     None

                model_config:  ClassVar[ConfigDict]  =  {}

                        Configuration for the model, should be a dictionary
                        conforming to
                        [ConfigDict][pydantic.config.ConfigDict].

   class  daisys.v1.speak.models.Webhook(*, post_url:  str, timestamp_ms: 
   int  |  None  =  None, status_code:  int  |  None  =  None)

           Store information about a registered webhook and its status.

           When specifying a webhook, only url needs to be provided.

                post_url

                        The URL to be called with POST.

                             Type:

                                     str

                timestamp_ms

                        The time it was last called at, milliseconds since
                        epoch.

                             Type:

                                     int | None

                status_code

                        The HTTP status code of the last response from the
                        webhook.

                             Type:

                                     int | None

                model_config:  ClassVar[ConfigDict]  =  {}

                        Configuration for the model, should be a dictionary
                        conforming to
                        [ConfigDict][pydantic.config.ConfigDict].

Daisys API websockets

   The Daisys API provides a websocket interface to enable direct
   communication with a single inference worker node, for applications that
   require lower latency.

  Latency vs. throughput

   While the websocket connection provides some convenience for certain
   applications, it should not be used for tasks where a batch approach is
   more appropriate, since generation requests through the REST API get
   distributed over multiple workers and will overall finish faster. However,
   applications that require real-time or near real-time interaction may
   benefit from keeping a connection open and receiving the response
   immediately without making an extra HTTP GET request.

   Please do keep in mind that an effect of dedicating a connection is that
   requests over this connection are effectively serialized, so the decision
   of using websocket vs. the REST API is a typical latency vs. throughput
   tradeoff to make. For this reason, to help guarantee latency, the
   websocket system reserves the right to occasionally drop the connection,
   forcing the client to request a new URL, which has the effect of
   rebalancing the distribution of connections to workers, helping to ensure
   lower latency overall.

   In this document we describe:

     * Connecting: How to get a websocket URL and make and maintain a
       connection.

     * Message format: The message format used to send commands and receive
       responses.

     * Python interface: How to use this Python library to communicate over
       the websocket.

   Examples of using the websocket connection from the Python API as well as
   communicating with the websocket from JavaScript in a browser application
   are given in Daisys API websocket examples.

  Connecting

   In order to connect to a Daisys worker node, you must first be assigned a
   node through the API. In Python, this is taken care of for you, see Python
   interface below. However, when using another language or curl, you can get
   a URL via the Websocket Endpoints using a GET request.

   As mentioned there, the websocket may disconnect between requests for
   rebalancing worker load, although this should not happen frequently.
   Additionally it is assumed that a websocket connection is for interaction
   with a specific model, which must be included in the URL. In fact any
   model can be used on a websocket connection but only the specified model
   shall be kept from being unloaded, therefore if latency is at issue, it is
   recommended to open a websocket connection per model. A reconnection
   scheme can be used to immediately request a new worker URL and reconnect
   if the connection is dropped. The Daisys API shall make every effort to
   ensure that all current requests are handled and results delivered before
   dropping any connections.

  Message format

   In cases where you are not using Python and wish to develop your own
   client for the websocket, the format is kept rather simple and should be
   quite approachable for any language for which a websocket library is
   available.

   Websocket supports text and bytes (binary) messages. Commands are sent
   using text messages, and status messages (text) and audio messages (bytes)
   are received. Both outgoing and incoming text messages are in JSON format.

   Outgoing messages have the following format:

 {"command": "<command>", "data": {<data>}, "request_id": <request_id>}

   where command may be one of /takes/generate or /voices/generate.

   The data field corresponds to the same POST body given to the
   corresponding commands, i.e. the TakeGenerate and VoiceGenerate
   structures, respectively.

   Likewise, the status messages received for each correspond to the
   responses to those same commands, these being TakeGenerate and
   TakeGenerate, respectively. They are similarly bundled into a response
   structure,

 {"data": {<data>}, "request_id": <request_id>}

   Special to the websocket connection is request_id, which is needed to
   track which incoming responses go with which outgoing requests. Because
   websockets do not guarantee message order (shorter messages may arrive
   before longer messages), and because a take_id is not known until the
   first status message is received, there is no way to know which audio goes
   with which request. Therefore the request_id is a user-provided
   identifier, a string or an integer, which is included with the responses
   to that command. A simple incrementing integer per connection is
   recommended, and is what the Python interface implements.

   Audio response messages are also simple, however since it is necessary to
   carry some metadata, they contain two sections, delimited by a length
   prefix. Audio messages (bytes) are formatted thus:

 JSON<length 4 bytes><json metadata>RIFF..

   That is, they start with the literal string JSON followed by a 32-bit
   little endian integer indicating how long the metadata section is. The
   metadata section can be converted to a string and parsed as JSON. This is
   immediately followed by a .wav file header, which always starts with the
   literal string RIFF. Therefore, starting at R, the rest of the bytes can
   be passed to an audio player or a wav file parsing routine if chunking is
   not used.

   The metadata section consists of the following fields,

 {"take_id": "<take_id>", "part_id": int, "chunk_id": int, "request_id": <request_id>}

   where part_id and chunk_id are incrementing integers as specified in the
   next section, and request_id reflects whatever was provided when the
   associated command was issued.

  Parts and chunks

   If multiple sentences have been provided, then they are returned with
   separate part_id values, which are an incrementing integer, where each
   part consists of a complete wav file. The end of the stream for a take is
   indicated by a new part_id that has 0 bytes of audio.

   If chunking is enabled, the bytes must be concatenated to an existing part
   stream, either in real time or before writing the part to a file. Chunks
   are different from parts in that they are not prepended with a wav header,
   but are merely the individual pieces of a part that is not yet fully
   received. Similar to parts, chunks are identified with an incrementing
   integer chunk_id which must be used to put them in order before playback.
   Also similar, the end of the chunk stream for a part is indicated by a new
   chunk_id being accompanied with 0 bytes of audio.

   Finally then, a stream of parts without chunking appears like so:

 [part_id=0, audio len=12340]
 [part_id=1, audio len=23450]
 [part_id=2, audio len=0]

   and with chunking,

 [part_id=0, chunk_id=0, audio len=4140]
 [part_id=0, chunk_id=1, audio len=4140]
 [part_id=0, chunk_id=2, audio len=0]
 [part_id=1, chunk_id=0, audio len=4140]
 [part_id=1, chunk_id=1, audio len=4140]
 [part_id=1, chunk_id=2, audio len=0]
 [part_id=2, chunk_id=0, audio len=0]

   The above is for visual explanation only, in reality the take_id and
   request_id are also included in the metadata header in order to know which
   audio is for which stream.

   I a /voices/generate message was requested, audio of the associated
   example take will be sent. However the status message will be a
   TakeGenerate object, and the take_id included in the audio messages will
   correspond with its example_take_id field.

  Python interface

   The Python interface consists of calling, websocket() of the client object
   (see Daisys API clients), in a with context manager, which returns
   respectively one of the following objects. For example,

   Streaming audio, websocket method

 from daisys import DaisysAPI
 with DaisysAPI('speak', email='user@example.com', password='pw') as speak:
     with speak.websocket(model='theatrical-v2') as ws:
         ....
         request_id = ws.generate_take(...

   In each case, you can then issue a command to generate a take or a voice
   using the returned context object, as demonstrated above. Subsequently the
   callbacks you provide get called whenever messages are received on the
   websocket containing either status information or audio data.

   class  daisys.v1.speak.sync_websocket.DaisysSyncSpeakWebsocketV1(client: 
   DaisysSyncSpeakClientV1, model:  str)

           Wrapper for Daisys v1 API websocket connection, synchronous
           version.

           This class is intended to be used in a with clause.

                disconnect()

                        Disconnect this websocket.

                generate_take(voice_id:  str, text:  str, override_language: 
                str  |  None  =  None, style:  list[str]  |  None  =  None,
                prosody:  SimpleProsody  |  AffectProsody  |  SignalProsody 
                |  None  =  None, stream_options:  StreamOptions  |  None  = 
                None, status_webhook:  str  |  None  =  None, done_webhook: 
                str  |  None  =  None, status_callback:  Callable[[int, 
                TakeResponse],  None]  |  None  =  None, audio_callback: 
                Callable[[int,  str,  int,  int  |  None,  bytes  |  None], 
                None]  |  None  =  None, timeout:  float  |  None  =  None) →
                int

                        Generate a “take”, an audio file containing an
                        utterance of the given text by the given voice.

                             Parameters:
                                        * voice_id – The id of the voice to
                                          be used for generating audio. The
                                          voice is attached to a specific
                                          model.

                                        * text – The text that the voice
                                          should say.

                                        * override_language – Normally a
                                          language classifier is used to
                                          detect the language of the speech;
                                          this allows for multilingual
                                          sentences. However, if the language
                                          should be enforced, it should be
                                          provided here. Currently accepted
                                          values are “nl-NL” and “en-GB”.

                                        * style – A list of styles to enable
                                          when speaking. Note that most
                                          styles are mutually exclusive, so a
                                          list of 1 value should be provided.
                                          Accepted styles can be retrieved
                                          from the associated voice’s
                                          VoiceInfo.styles or the model’s
                                          TTSModel.styles field. Note that
                                          not all models support styles, thus
                                          this can be left empty if specific
                                          styles are not desired.

                                        * prosody – The characteristics of
                                          the desired speech not determined
                                          by the voice or style. Here you can
                                          provide a SimpleProsody or most
                                          models also accept the more
                                          detailed AffectProsody.

                                        * stream_options – Configuration for
                                          streaming.

                                        * status_webhook – An optional URL to
                                          be called using POST whenever the
                                          take’s status changes, with
                                          TakeResponse in the body content.

                                        * done_webhook – An optional URL to
                                          be called exactly once using POST
                                          when the take is READY, ERROR, or
                                          TIMEOUT, with TakeResponse in the
                                          body content.

                                        * status_callback – An optional
                                          function to call for status updates
                                          regarding this take.

                                        * audio_callback – An optional
                                          function to call to provide the
                                          audio parts of the take.

                             Returns:

                                     Information about the take being
                                     generated, including status.

                             Return type:

                                     TakeResponse

                generate_voice(name:  str, model:  str, gender:  VoiceGender,
                description:  str  |  None  =  None, default_style: 
                list[str]  |  None  =  None, default_prosody:  SimpleProsody 
                |  AffectProsody  |  SignalProsody  |  None  =  None,
                example_take:  TakeGenerateWithoutVoice  |  None  =  None,
                stream_options:  StreamOptions  |  None  =  None,
                done_webhook:  str  |  None  =  None, status_callback: 
                Callable[[int,  TakeResponse],  None]  |  None  =  None,
                audio_callback:  Callable[[int,  str,  int,  int  |  None, 
                bytes  |  None],  None]  |  None  =  None) → int

                        Generate a random, novel voice for a given model with
                        desired properties.

                             Parameters:
                                        * name – A name to give the voice,
                                          may be any string, and does not
                                          need to be unique.

                                        * model – The name of the model for
                                          this voice.

                                        * gender – The gender of this voice.

                                        * description – The description of
                                          this voice.

                                        * default_style – An optional list of
                                          styles to associate with this voice
                                          by default. It can be overriden by
                                          a take that uses this voice. Note
                                          that most styles are mutually
                                          exclusive, and not all models
                                          support styles.

                                        * default_prosody – An optional
                                          default prosody to associate with
                                          this voice. It can be overridden by
                                          a take that uses this voice.

                                        * example_take – Information on the
                                          take to generate as an example of
                                          this voice.

                                        * stream_options – Configuration for
                                          streaming.

                                        * done_webhook – An optional URL to
                                          call exactly once using POST when
                                          the voice is available, with
                                          VoiceInfo in the body content.

                                        * status_callback – An optional
                                          function to call for status updates
                                          regarding this take.

                                        * audio_callback – An optional
                                          function to call to provide the
                                          audio parts of the take.

                             Returns:

                                     Information about the generated voice.

                             Return type:

                                     VoiceInfo

                iter_request(request_id)

                        Iterate over incoming text and audio messages for a
                        given request_id.

                             Parameters:

                                     request_id – The id value associated
                                     with the request to be iterated over.
                                     Returned by take_generate() and
                                     voice_generate().

                             Returns:

                                     An Iterator yielding tuples (take_id,
                                     take, header, audio), where:

                                        * take_id: the take_id associated
                                          with this request

                                        * take: the TakeResponse information
                                          if a text message, otherwise None

                                        * header: the wav header if any,
                                          otherwise None

                                        * audio: the audio bytes, if a binary
                                          message, otherwise None

                reconnect()

                        Reconnect this websocket, by first fetching the URL
                        and then opening the conneciton to it.

                update(timeout:  int  |  None  =  1)

                        Retrieve a waiting message on the open websocket
                        connection.

                             Parameters:

                                     timeout – Number of seconds to wait. Can
                                     be 0 if non-blocking usage is desired.
                                     If None, wait forever.

   class  daisys.v1.speak.async_websocket.DaisysAsyncSpeakWebsocketV1(client:
   DaisysAsyncSpeakClientV1, model:  str  |  None, voice_id:  str  |  None)

           Wrapper for Daisys v1 API websocket connection, asynchronous
           version.

           This class is intended to be used in an async with clause.

                async  disconnect()

                        Disconnect this websocket.

                async  generate_take(voice_id:  str, text:  str,
                override_language:  str  |  None  =  None, style:  list[str] 
                |  None  =  None, prosody:  SimpleProsody  |  AffectProsody 
                |  SignalProsody  |  None  =  None, stream_options: 
                StreamOptions  |  None  =  None, status_webhook:  str  | 
                None  =  None, done_webhook:  str  |  None  =  None,
                status_callback:  Callable[[int,  TakeResponse],  None]  | 
                None  =  None, audio_callback:  Callable[[int,  str,  int, 
                int  |  None,  bytes  |  None],  None]  |  None  =  None,
                timeout:  float  |  None  =  None) → int

                        Generate a “take”, an audio file containing an
                        utterance of the given text by the given voice.

                             Parameters:
                                        * voice_id – The id of the voice to
                                          be used for generating audio. The
                                          voice is attached to a specific
                                          model.

                                        * text – The text that the voice
                                          should say.

                                        * override_language – Normally a
                                          language classifier is used to
                                          detect the language of the speech;
                                          this allows for multilingual
                                          sentences. However, if the language
                                          should be enforced, it should be
                                          provided here. Currently accepted
                                          values are “nl-NL” and “en-GB”.

                                        * style – A list of styles to enable
                                          when speaking. Note that most
                                          styles are mutually exclusive, so a
                                          list of 1 value should be provided.
                                          Accepted styles can be retrieved
                                          from the associated voice’s
                                          VoiceInfo.styles or the model’s
                                          TTSModel.styles field. Note that
                                          not all models support styles, thus
                                          this can be left empty if specific
                                          styles are not desired.

                                        * prosody – The characteristics of
                                          the desired speech not determined
                                          by the voice or style. Here you can
                                          provide a SimpleProsody or most
                                          models also accept the more
                                          detailed AffectProsody.

                                        * stream_options – Configuration for
                                          streaming.

                                        * status_webhook – An optional URL to
                                          be called using POST whenever the
                                          take’s status changes, with
                                          TakeResponse in the body content.

                                        * done_webhook – An optional URL to
                                          be called exactly once using POST
                                          when the take is READY, ERROR, or
                                          TIMEOUT, with TakeResponse in the
                                          body content.

                                        * status_callback – An optional
                                          function to call for status updates
                                          regarding this take.

                                        * audio_callback – An optional
                                          function to call to provide the
                                          audio parts of the take.

                             Returns:

                                     Information about the take being
                                     generated, including status.

                             Return type:

                                     TakeResponse

                async  generate_voice(name:  str, model:  str, gender: 
                VoiceGender, description:  str  |  None  =  None,
                default_style:  list[str]  |  None  =  None, default_prosody:
                SimpleProsody  |  AffectProsody  |  SignalProsody  |  None  =
                None, example_take:  TakeGenerateWithoutVoice  |  None  = 
                None, stream_options:  StreamOptions  |  None  =  None,
                done_webhook:  str  |  None  =  None, status_callback: 
                Callable[[int,  TakeResponse],  None]  |  None  =  None,
                audio_callback:  Callable[[int,  str,  int,  int  |  None, 
                bytes  |  None],  None]  |  None  =  None) → int

                        Generate a random, novel voice for a given model with
                        desired properties.

                             Parameters:
                                        * name – A name to give the voice,
                                          may be any string, and does not
                                          need to be unique.

                                        * model – The name of the model for
                                          this voice.

                                        * gender – The gender of this voice.

                                        * description – The description of
                                          this voice.

                                        * default_style – An optional list of
                                          styles to associate with this voice
                                          by default. It can be overriden by
                                          a take that uses this voice. Note
                                          that most styles are mutually
                                          exclusive, and not all models
                                          support styles.

                                        * default_prosody – An optional
                                          default prosody to associate with
                                          this voice. It can be overridden by
                                          a take that uses this voice.

                                        * example_take – Information on the
                                          take to generate as an example of
                                          this voice.

                                        * stream_options – Configuration for
                                          streaming.

                                        * done_webhook – An optional URL to
                                          call exactly once using POST when
                                          the voice is available, with
                                          VoiceInfo in the body content.

                                        * status_callback – An optional
                                          function to call for status updates
                                          regarding this take.

                                        * audio_callback – An optional
                                          function to call to provide the
                                          audio parts of the take.

                             Returns:

                                     Information about the generated voice.

                             Return type:

                                     VoiceInfo

                async  iter_request(request_id)

                        Iterate over incoming text and audio messages for a
                        given request_id.

                             Parameters:

                                     request_id – The id value associated
                                     with the request to be iterated over.
                                     Returned by take_generate() and
                                     voice_generate().

                             Returns:

                                     An Iterator yielding tuples (take_id,
                                     take, header, audio), where:

                                        * take_id: the take_id associated
                                          with this request

                                        * take: the TakeResponse information
                                          if a text message, otherwise None

                                        * header: the wav header if any,
                                          otherwise None

                                        * audio: the audio bytes, if a binary
                                          message, otherwise None

                async  reconnect()

                        Reconnect this websocket, by first fetching the URL
                        and then opening the conneciton to it.

                async  update(timeout:  int  |  None  =  1)

                        Retrieve a waiting message on the open websocket
                        connection.

                             Parameters:

                                     timeout – Number of seconds to wait. In
                                     the async implementation this cannot be
                                     0. If None, wait forever.

Daisys API endpoints

   While Daisys recommends the use of the Python client, the Daisys API
   endpoints are available for use with other languages. In addition to the
   current document, the FastAPI-generated documentation is available:

       * Swagger UI: https://api.daisys.ai/v1/speak/docs

       * Redoc: https://api.daisys.ai/v1/speak/redoc

       * OpenAPI definition file: https://api.daisys.ai/v1/speak/openapi.json

   See also the FastAPI documentation on how to generate clients for other
   languages.

   The “Speak” API provides a REST interface to its three main data
   structures: models, voices, and takes.

   This is best demonstrated in the curl example, where JSON objects are
   constructed as strings in a shell script. See JSON input structures for
   more information on JSON input.

  Model-related Endpoints

   A Daisys API user account may have access to one or more models. These
   models can be listed by accessing the models endpoint using a GET request:

 https://api.daisys.ai/v1/speak/models

   Furthermore a specific model can be accessed by providing its name:

 https://api.daisys.ai/v1/speak/models/<model_name>

  Voice-related Endpoints

   A Daisys API user account may have access to one or more voices. These
   voices can be listed by accessing the voices endpoint using a GET request.
   Voice listing can also be filtered, by providing fields voice_id (a
   comma-separated list of voice_id to retrieve), length, page, older and
   newer. In these case a list is returned:

 https://api.daisys.ai/v1/speak/voices
 https://api.daisys.ai/v1/speak/voices?voice_id=<voice_id1,voice_id2>
 https://api.daisys.ai/v1/speak/voices?length=5&page=2
 https://api.daisys.ai/v1/speak/voices?newer=1690214050638

   The argument for newer and older must be a timestamp in milliseconds since
   epoch. This can be retrieved in Python for example using:

 import time
 def seconds_ago(seconds: int=2):
     return int((time.time() - seconds)*1000)
 response = requests.get(f'https://api.daisys.ai/v1/speak/voices?newer={seconds_ago(2)}',
                         header={'Authorization', 'Bearer ' + access_token)

   Furthermore a specific voice can be accessed by providing its name. In
   this case a single item is returned instead of a list:

 https://api.daisys.ai/v1/speak/voices/<voice_id>

   User accounts may also have access to generate new voices. This can be
   done by making a POST request to the voices/generate endpoint:

 https://api.daisys.ai/v1/speak/voices/generate

   The body should contain the VoiceGenerate structure in JSON format.
   Example:

 curl -X POST -H "Authorization: Bearer $TOKEN" -H 'content-type: application/json' \
   -d '{"name": "Bob", "gender": "male", "model": "my_model"}' \
   https://api.daisys.ai/v1/speak/voices/generate

   where my_model should be the name of a model listed by the /speak/models
   endpoint.

  Take-related Endpoints

   The principle service of the Daisys API is to perform text-to-speech audio
   synthesis. This is done by generating “takes”, which encapsulate a TTS
   job. Previously generated takes can be retrieved via takes, and the list
   can be filtered similar to voices:

 https://api.daisys.ai/v1/speak/takes
 https://api.daisys.ai/v1/speak/takes?take_id=<take_id1,take_id2>
 https://api.daisys.ai/v1/speak/takes?length=5&page=2
 https://api.daisys.ai/v1/speak/takes?newer=1690214050638

   with similar semantics to /speak/voices described above. A single take can
   be retrieved by giving its identifier:

 https://api.daisys.ai/v1/speak/takes/<take_id>

   An audio take can be generated by making a POST request to takes/generate:

 https://api.daisys.ai/v1/speak/takes/generate

   and providing the TakeGenerate structure as input in the content body.

   Finally, the audio can be retrieved by accessing the take’s /wav endpoint.
   Equivalently, other formats can also be retrieved this way, however wav is
   the only format that can be retrieved before it is “ready”, allowing to
   download as it is generated:

 https://api.daisys.ai/v1/speak/takes/<take_id>/wav
 https://api.daisys.ai/v1/speak/takes/<take_id>/mp3
 https://api.daisys.ai/v1/speak/takes/<take_id>/m4a
 https://api.daisys.ai/v1/speak/takes/<take_id>/flac
 https://api.daisys.ai/v1/speak/takes/<take_id>/webm

   Note that these endpoints return a 307 redirect to where the audio can be
   streamed or stored from.

     Important: a complication is that S3 presigned URLs must be accessed
     without the Daisys “Authorization” header, which some http clients will
     not drop automatically. Therefore the following logic is recommended,
     and performed by the Python client library when following the redirect
     to url:

 if 'X-Amz-Signature' in url:
   # Pre-signed URL, no auth needed.
   headers = {}

     Note that browsers handle this automatically when changing origins,
     however it is not recommended in any case to access the REST API
     endpoints directly from the browser since they require the access token.
     Instead, backend software can access the /wav endpoint and retrieve the
     URL in the Location header, and forward this to the browser, which can
     be access without the Authorization header and has a limited lifetime.
     Therefore this redirect Location is convenient and more secure to pass
     directly to an Audio Player object on the client side.

    Retrieving audio

   Finally, the audio can be retrieved by accessing the take’s /wav endpoint.
   Equivalently, other formats can also be retrieved this way, however wav is
   the only format that can be retrieved before it is “ready”, allowing to
   download as it is generated:

 https://api.daisys.ai/v1/speak/takes/<take_id>/wav
 https://api.daisys.ai/v1/speak/takes/<take_id>/mp3
 https://api.daisys.ai/v1/speak/takes/<take_id>/m4a
 https://api.daisys.ai/v1/speak/takes/<take_id>/flac
 https://api.daisys.ai/v1/speak/takes/<take_id>/webm

   Note that these endpoints return a 307 redirect to where the audio can be
   streamed or stored from.

     Important: a complication is that S3 presigned URLs must be accessed
     without the Daisys “Authorization” header, which some http clients will
     not drop automatically. Therefore the following logic is recommended,
     and performed by the Python client library when following the redirect
     to url:

 if 'X-Amz-Signature' in url:
   # Pre-signed URL, no auth needed.
   headers = {}

     Note that browsers handle this automatically when changing origins,
     however it is not recommended in any case to access the REST API
     endpoints directly from the browser since they require the access token.
     Instead, backend software can access the /wav endpoint and retrieve the
     URL in the Location header, and forward this to the browser, which can
     be accessed without the Authorization header and has a limited lifetime.
     Therefore this redirect Location is convenient and more secure to pass
     directly to an Audio Player object on the client side.

  Websocket Endpoints

   The following endpoint can be used to retrieve an URL for making a direct
   websocket connection to a worker by issuing a GET request:

 https://api.daisys.ai/v1/speak/websocket?model=<model>

   As can be seen, the model to use must be specified when making a request
   for a worker URL, which allows the Daisys API to better distribute
   requests to workers with preloaded models.

   For the same reason, whenever a websocket is disconnected, a new URL must
   be requested through the above endpoint. Disconnection may happen from
   time to time but shall not happen during the processing of a request. The
   provided URLs expire after 1 hour. A connection may remain open longer
   than that, but new connections must request a new URL.

   The endpoint returns the following JSON body:

 {
   "websocket_url": "<url>"
 }

  Authentication Endpoints

   To make use of the Daisys API, first an access token must be granted. This
   can be retrieved by a POST request to the auth/login endpoint:

 https://api.daisys.ai/auth/login

   The content body should have the form:

 {
   "email": <user@example.com>,
   "password": <password>
 }

   On failure, a 401 HTTP status is returned. (In the client library, an
   exception is raised.) On success, a JSON object containing access_token
   and refresh_token fields is provided.

   The access_token string should be attached to all GET and POST requests in
   the HTTP header, in the following form:

 Authorization: Bearer <access_token>

   Furthermore if the access_token is no longer working, the refresh_token
   can be used to get a new one without supplying the password:

 https://api.daisys.ai/auth/refresh

   In this case the POST request should have the form:

 {
   "email": <user@example.com>,
   "refresh_token": <refresh_token>
 }

   The response contains new access_token and refresh_token fields. This
   allows to continually refresh an initial token whenever needed, so that
   the API can be used without providing a password.

   Note that this token refresh logic is taken care of automatically by the
   Python client library. The client can also be initiated with just an email
   and refresh token rather than an email and password, so that credentials
   need not be provided to the Daisys API client. It is also alternatively
   possible to request a permatoken, which does not need to be refreshed.

   On the other hand, refresh tokens can be revoked at any time through the
   following POST endpoint:

     https://api.daisys.ai/auth/logout

   with content body of the form:

 {
   "refresh_token": <refresh_token>,
 }

  JSON input structures

   POST endpoints, namely takes/generate and voices/generate, take input in
   their content body in the form of JSON objects.

   The structure of all such objects can be inferred by reading the models,
   since the fields can be translated directly to JSON. Nonetheless some of
   the embedded structures and optional fields can be confusing, thus we give
   some examples here.

   A minimal example of TakeGenerate:

 {
   "text": "This is some text to speak.",
   "prosody": {"pace": -3, "pitch": 0, "expression": 4},
   "voice_id": "01h3anwqdh1q6zhf9s9s239wky",
 }

   Optional fields such as style, override_language, and done_webhook can be
   added as desired.

   Here is an example of TakeGenerate using all available fields:

 {
   "text": "This is some text to speak.",
   "override_language": "en-GB",
   "prosody": {"pace": -3, "pitch": 0, "expression": 4},
   "voice_id": "01h3anwqdh1q6zhf9s9s239wky",
   "style": ["narrator"],
   "status_webhook": "https://myservice.com/daisys_webhooks/take_status/1234",
   "done_webhook": "https://myservice.com/daisys_webhooks/take_done/1234",
 }

   Note that override_language is provided here as an example, but if it is
   not provided (is null) then the Daisys API will attempt to pronounce words
   in the correct language on a per-word basis. If it is provided, then the
   model may for example mispronounce loan words, since it assumes a single
   language for the input text. The presence of the style field depends on
   the model in use, as does the supported prosody types, although all models
   support the simple prosody type with pace, pitch, and expression being
   integer values from -10 to 10. Specific information about the model can be
   retrieved by the /speak/models endpoint.

   Finally, here is an example of input for voices/generate:

 {
   "name": "Bob",
   "default_prosody": {"pace": 0, "pitch": 0, "expression": 0},
   "model": "eng_base",
   "gender": "male",
   "done_webhook": "https://myservice.com/daisys_webhooks/voice_done/1234",
 }

   Here, a default prosody is specified for the voice, which is adopted in
   subsequent /take/generate requests if prosody is not provided (left as
   null).

                              Indices and tables

     * Index

     * Module Index

     * Search Page

                                   daisys-api

  Navigation

   Contents:

     * Getting started
     * Daisys API examples
     * Daisys API input
     * Daisys API top-level object
     * Daisys API clients
     * Daisys API JSON models
     * Daisys API websockets
     * Daisys API endpoints

  Related Topics

     * Documentation overview
   ©2025, Daisys AI. | Powered by Sphinx 8.1.3 & Alabaster 1.0.0