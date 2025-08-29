# langgraph_generative_ui


# langgraph_fastapi_server
langgraph app  serve using fastapi server


#use case :
generative ui provide good user visualise data and text in ui format
for code generation task code stream in code ui
weather and news provide in intractive ui
data represation in chart and table.


how it works :
[1].at backend when in invoke graph  ,for example  React agent tool call ,agent select tool calling based on user query then tool_node executing , tool node
streaming AIMessage/ToolMessage with name of tool , (for streaming/partial response in ui AIMessage/ToolMessage(id=const) ), so when client receive Messages based on  messages.name componet render and additional_kwargs data and content to generate ui component informations.

🔴alert: In ToolMessage tool_call_id use for llm to recognize which ToolMessage belong to AI tool_calls request , it necessary to provide make  ToolMessage(),
(tool_calls contain tool_call_id it pass to ToolMessage(tool_call_id="provide_by_llm") otherwise llm wrong interprete which tool_calls belong to  ToolMessage answer. 

details explain  : how it work
 [1]. client(Reactjs app) and server    
 
>client use EventSource() method to request server with input and checkpoint_id (here checkpoint_id use for persist state in  langgraph app across multiple invoke )
>server get request with input and checkpoint_id ( based on checkpoint_id graph invoke and yield to StreamingResponse() )
>server response using StreamingRespose() sse event (response format is    "data:{"type":"chunk"/"content"/"end","content":"AIMessage" or "ToolMessage"}"
>client get reponse back from server.

[2]. Render message and ui based on  Message type and id 
>when client start new conversation (checkpoint_id=None) then server send checkpoint id to client (or client also request with checkpoint_id to persist conversation or start new conversation)
>client get main two type of stream messages 
1.chunk:custom/llm chunk(from get_stream_writer()/llm ) (stream_mode=['custom','messages])
2.content:client receive updated state after every node executed in graph (stream_mode=['values'])
>setMessage for stream chunk and setState for stream state from server
>client render message and ui based on message type(based on message type we render ui) and id(for streaming chunk and partial response from agent)
>(important) client receive streaming chunk and aggregate to stream chunk then render in  bubble  and after get stream state , chunk message is flush render and  state messages


[3]. graph run type   output = graph.stream(input , config, stream_mode )/.astream()/.astream_events()   
>diffrent graph run method give diffrent output
>stream_mode =['custom','values','messages','update' ,'debug']


[4] graph.stream()/astream()/astream_events() run method support stream_mode:
>'messages', 'custom' mode is use for streaming ouput chunk from graph  (chunk from inside graph node)
>'values','update' mode streaming state at itermeadiate graph step (when node return state )  step  

[5]graph.invoke()     // not use when require real time stream
>graph output state end of graph


[6].graph node
>get_stream_writer() method play a important role in Generative ui, custom message stream (partial_response )
>get_stream_writer() use for stream custom  chunk , after stream complate aggregate custom_chunk return ,so state contain messages  (help full for frontend setState for render ui atfer streaming end )
>get_stream_writer() work with stream_mode='custom'
>langchain LCEL chain or langchain llm can streaming chunk when stream_mode='messages
