# langgraph_generative_ui


# langgraph_fastapi_server
langgraph app  serve using fastapi server




for start application run locally
pull repo 
cd ./langgraph_fastAPI_server
1.pip install -e .
2.create .env file contains google_api_key and tavily_api_key
3.uvicorn app:app       //run  awsgi server

 

how it works 

 [1]. client(Reactjs app) and server    
 
>client use EventSource() method to request server with input and checkpoint_id (here checkpoint_id use for persist state in  langgraph app across multiple invoke )
>server get request with input and checkpoint_id ( based on checkpoint_id graph invoke and yield to StreamingResponse() )
>server response using StreamingRespose() sse event (response format is    "data:{"type":"chunk"/"content"/"end","content":"AIMessage" or "ToolMessage"}"
>client get reponse back from server.

[2]. Render message and ui based on  Message type and id 
>setMessage for stream chunk and setState for stream state from server
>client render message and ui based on message type(based on message type we render ui) and id(for streaming chunk and partial response from agent)
>(important) client  render streaming chunk to bubble and after get state render and setMessage("") flush , and setState() render to ui


[3]. graph run type   output = graph.stream(input , config, stream_mode )/.astream()/.astream_events()   
>diffrent graph run method give diffrent output
>stream_mode =['custom','values','messages','update' ,'debug']


[4] graph.stream()/astream()/astream_events() run method support stream_mode:
>'messages', 'custom' mode is use for streaming ouput chunk from graph  (chunk from inside graph node)
>'values','update' mode streaming state at itermeadiate graph step (when node return state )  step  

[5]graph.invoke()     // not use when need real time stream
>graph output state end of graph


[6].graph node
>get_stream_writer() method play a important role in Generative ui, custom message stream (partial_response )
>get_stream_writer() use for stream custom  chunk , after stream complate aggregate custom_chunk return ,so state contain messages  (help full for frontend setState for render ui atfer streaming end )
>get_stream_writer() work with stream_mode='custom'
>langchain LCEL chain or langchain llm can streaming chunk when stream_mode='messages
