# E D B G

A tty based interface to the Erlang debugger and tracer.

## Table of Contents
1. [Install](#install)
2. [Debug Usage](#dbg-usage)
3. [Trace Usage](#trace-usage)
3. [Debug examples](#dbg-examples)
4. [Trace examples](#trace-examples)

Useful if you, for example, work from home but still
want to debug your code at your work desktop, or if you
simply don't like the standard debugger GUI.

Note that lots of the debugger functionality is already supported by the
standard OTP `int.erl` module. What the edbg debugger brings is basically the
tty based attached mode and the possibility to make use of interactively
defined conditional break points.

The edbg tracing functionality is (hopefully) a somewhat more novel approach.

<a name="install"></a>
## INSTALL
```
   Run: make
   Add: code:add_path("YOUR-PATH-HERE/edbg/ebin").
   Add: code:add_path("YOUR-PATH-HERE/edbg/deps/pp_record/ebin").
     to your ~/.erlang file.
```

<a name="dbg-usage"></a>
## DEBUG USAGE

### edbg:i(Mod, Line)
Start interpret module `Mod` and set a break point at line `Line`.

### edbg:b(Mod, Line) | edbg:break(Mod, Line)
Set a break point in `Mod` at line `Line`.

### edbg:it(Mod, Line, Fun)
Start interpret module `Mod` and set a conditional break point
 in `Mod` at line `Line`.
The `Fun/1` as an anonymous function of arity 1 that gets executed
each time the break point is passed. When the `Fun/1` returns
`true` the break point will trigger and the execution will stop,
else the execution will continue.

The `Fun/1` takes an argument which will be a list of current Variable
bindings; typically it makes use of the function
`int:get_binding(Var, Bindings)` (where `Var` is an atom denoting a
particular variable) to decide if the break point should trigger
or not. See the example further below for how to use it.

Note that only one interactive trigger function can be used at a time.

### edbg:t(Mod, Line) | edbg:t(Mod, Line, Fun)
Works at `edbg:it/3` but assumes that the module alread is being
interpreted.

The `t/2` function will reuse an already existing trigger function.

### edbg:pl() | edbg:plist()
Show all interpreted processes and what state there are in.
In particular this is useful to see if a process is has stopped
in a break point. The process identifier (Pid) displayed in the
leftmost column can be used with the `edbg:attach/1` function.

### edbg:a(Pid) | edbg:attach(Pid)
### edbg:a(P0, P1, P2) | edbg:attach(P0, P1, P2)
### edbg:a()
Attach to an interpreted process in order to manually control
the further execution, inspect variables, etc. When called, you
will enter a sort of mini-shell where you can issue a number of
commands; see the examples below.

The `a/0` function is a shortcut to attach to the first process found
that has stopped at a break point.

The `a/3` and `attach/3` functions are shorthands for supplying a pid as a
sequence of integers.

### edbg:mlist(Mod) | edbg:mlist(Pid) | edbg:mlist(Mod, Line) | edbg:mlist(Mod, Line, Context) | edbg:mlist(P0, P1, P2)
### edbg:ml(Mod) | edbg:ml(Pid) | edbg:ml(Mod, Line) | edbg:ml(Mod, Line, Context)
List the source code of a Module, either centered around a triggered
break point, or around a given `Line`. The amount of lines being display
around the line is controled by the `Context` value, which per default
is set to `5` (i.e display 5 lines above and below the line).

Note that the listing will display the line numbers at the left border
of the output where breakpoints are high lighted by a `*` character
and the given line as `>`. However, if no line is given, the `>`
character will be used to denote where we currently are stopped.

The `mlist/3` can take a sequence of integers for supplying a pid.

### edbg:br() | edbg:breaks()
Display all break points.

### edbg:br(Mod) | edbg:breaks(Mod)
Display the break points in module `Mod`.

### edbg:s(Pid) | edbg:step(Pid)
### edbg:s(P0, P1, P2) | edbg:step(P0, P1, P2)
Step the execution of a stopped process.

### edbg:n(Pid) | edbg:next(Pid)
### edbg:n(P0, P1, P2) | edbg:next(P0, P1, P2)
Next the execution of a stopped process.

### edbg:c(Pid) | edbg:continue(Pid)
### edbg:c(P0, P1, P2) | edbg:continue(P0, P1, P2)
Continue the execution of a stopped process.

### edbg:id()
Show all interpreted modules.

### edbg:delete_break(Mod,Line) | edbg:disable_break(Mod,Line) | edbg:enable_break(Mod,Line)
Delete/Disable/Enable a break point.

### edbg:load_all_breakpoints() | edbg:lab()
Whenever a break point is set or modified, information is
stored on disk in the file `breakpoints.edbg`. This function
will load and set those breakpoints found in this file.

<a name="trace-usage"></a>
## TRACE USAGE

### edbg:tstart(Mod, Mods [,Opts])
Start tracing the first time `Mod` (an atom) is called, in any process,
and in any spawned process from such a process.
Do also trace any call to the modules `Mods` (list of atoms).

`Opts` is a list of any of the following options:

* `{setup_mod, Mod}` : the module that will setup the tracing; default is `edbg_trace_filter`.
* `{trace_max}` : max number of trace messages; default is 10000.

The trace-start setup will automatically be stored on file
for easy reload.

### edbg:tlist()
Enter trace list mode, where the trace output can be inspected.

### edbg:tquit()
Stop and quit the edbg tracer.

### edbg:lts()
Load the latest trace-start setup from file.


<a name="dbg-examples"></a>
## DEBUG EXAMPLES
```erlang
   # LOAD THE yaws_server MODULE INTO THE DEBUGGER AND SET A BREAK POINT
   1> edbg:i(yaws_server, 1189).
   ok

   # NOW RUN THE TEST...

   # LIST DEBUGGED PROCESSES
   2> edbg:plist().
   <0.536.0>  yaws_server:gserv_loop/4   waiting
   <0.648.0>  yaws_server:acceptor0/2    break       yaws_server:1189
   <0.537.0>  yaws_server:POST/4         exit        Reason: shutdown
   <0.516.0>  yaws_server:safe_decode_p  idle
   <0.663.0>  yaws_server:acceptor0/2    running
   ok

   # ATTATCH TO A DEBUGGED PROCESS
   # NOTE THE LINE WHERE WE HAVE STOPPED ON THE BREAK POINT: '1189>'
   3> edbg:attach(0,648,0).

    (h)elp (a)t <Ctx> (n)ext (s)tep (f)inish (c)ontinue s(k)ip
    (m)essages (t)oggle trace (q)uit (br)eakpoints (p)rocesses
    (de)lete/(di)sable/(en)able/(te)st/(b)reak <Line | Mod Line>
    (v)ar <variable> (e)val <expr> (i)nterpret <Mod>  conte(x)t <Ctx>
    (u)p (d)own (l)ist module <Mod Line <Ctx>>
     MODULE: yaws_server.erl
     1184:         _ ->
     1185:             ok
     1186:     end,
     1187:
     1188:     process_flag(trap_exit, false),
     1189>     init_db(),
     1190:     SSL = GS#gs.ssl,
     1191:     Head = yaws:http_get_headers(CliSock, SSL),
     1192:     process_flag(trap_exit, true),
     1193:     ?Debug("Head = ~p~n", [Head]),
     1194:     case Head of

   # NOW WE CAN SINGLE STEP AS USUAL
   # NOTE THE LINE INDICATION FOR A BREAK POINT: '1189*'
   (<0.648.0>)> n
   MODULE: yaws_server.erl
   1185:             ok
   1186:     end,
   1187:
   1188:     process_flag(trap_exit, false),
   1189*     init_db(),
   1190>     SSL = GS#gs.ssl,
   1191:     Head = yaws:http_get_headers(CliSock, SSL),
   1192:     process_flag(trap_exit, true),
   1193:     ?Debug("Head = ~p~n", [Head]),
   1194:     case Head of
   1195:         {error, {too_many_headers, ReqTooMany}} ->

   # WE STEP ONE MORE LINE AND CHECK THE CONTENT OF A VARIABLE
   # NOTE THAT WE ONLY NEED TO SPECIFY A PREFIX OF THE VARIABLE NAME
   (<0.648.0>)> n
   MODULE: yaws_server.erl
   1186:     end,
   1187:
   1188:     process_flag(trap_exit, false),
   1189*     init_db(),
   1190:     SSL = GS#gs.ssl,
   1191>     Head = yaws:http_get_headers(CliSock, SSL),
   1192:     process_flag(trap_exit, true),
   1193:     ?Debug("Head = ~p~n", [Head]),
   1194:     case Head of
   1195:         {error, {too_many_headers, ReqTooMany}} ->
   1196:             %% RFC 6585 status code 431

   (<0.648.0>)> v SS
   'SSL' = nossl

   # IF WE KNOW THE VARIABLE CONTAINS A RECORD, WE CAN PRETTY PRINT IT
   (<0.660.0>)> pr GS
   'GS' =
   #gs{gconf =
           #gconf{
               yaws_dir = undefined,trace = false,flags = 64,
               logdir = "./logs",ebin_dir = [],src_dir = [],runmods = [],
       ....


   # AFTER SOME MORE 'next' OPERATIONS, WE ARE HERE:
   (<0.648.0>)> n
   MODULE: yaws_server.erl
   1198:             SC = pick_sconf(GS#gs.gconf, #headers{}, GS#gs.group),
   1199:             put(sc, SC),
   1200:             put(outh, #outh{}),
   1201:             deliver_431(CliSock, ReqTooMany);
   1202:         {Req0, H0} when Req0#http_request.method /= bad_request ->
   1203>             {Req, H} = fix_abs_uri(Req0, H0),
   1204:             ?Debug("{Req, H} = ~p~n", [{Req, H}]),
   1205:             SC = pick_sconf(GS#gs.gconf, H, GS#gs.group),
   1206:             put(outh, #outh{}),
   1207:             put(sc, SC),
   1208:             DispatchResult = case SC#sconf.dispatch_mod of

   # WE CAN NOW STEP INTO A FUNCTION AS USUAL
   (<0.648.0>)> s
   MODULE: yaws_server.erl
   91420: deepforeach(F, X) ->
   1421:     F(X).
   1422:
   1423:
   1424: fix_abs_uri(Req, H) ->
   1425>     case Req#http_request.path of
   1426:         {absoluteURI, _Scheme, Host0, Port, RawPath} ->
   1427:             Host = case Port of
   1428:                        P when is_integer(P) ->
   1429:                            Host0 ++ [$: | integer_to_list(P)];
   1430:                                                 % Is this ok?

   # WE CAN GO UP IN THE CALL CHAIN...
   (<0.648.0>)> u
   MODULE: yaws_server.erl
   1198:             SC = pick_sconf(GS#gs.gconf, #headers{}, GS#gs.group),
   1199:             put(sc, SC),
   1200:             put(outh, #outh{}),
   1201:             deliver_431(CliSock, ReqTooMany);
   1202:         {Req0, H0} when Req0#http_request.method /= bad_request ->
   1203>             {Req, H} = fix_abs_uri(Req0, H0),
   1204:             ?Debug("{Req, H} = ~p~n", [{Req, H}]),
   1205:             SC = pick_sconf(GS#gs.gconf, H, GS#gs.group),
   1206:             put(outh, #outh{}),
   1207:             put(sc, SC),
   1208:             DispatchResult = case SC#sconf.dispatch_mod of


   # ...AND DOWN AGAIN
   (<0.648.0>)> d
   MODULE: yaws_server.erl
   1420: deepforeach(F, X) ->
   1421:     F(X).
   1422:
   1423:
   1424: fix_abs_uri(Req, H) ->
   1425>     case Req#http_request.path of
   1426:         {absoluteURI, _Scheme, Host0, Port, RawPath} ->
   1427:             Host = case Port of
   1428:                        P when is_integer(P) ->
   1429:                            Host0 ++ [$: | integer_to_list(P)];
   1430:                                                 % Is this ok?

   # WE CAN FINISH THIS FUNCTION
   (<0.648.0>)> f
   MODULE: yaws_server.erl
   1199:             put(sc, SC),
   1200:             put(outh, #outh{}),
   1201:             deliver_431(CliSock, ReqTooMany);
   1202:         {Req0, H0} when Req0#http_request.method /= bad_request ->
   1203:             {Req, H} = fix_abs_uri(Req0, H0),
   1204>             ?Debug("{Req, H} = ~p~n", [{Req, H}]),
   1205:             SC = pick_sconf(GS#gs.gconf, H, GS#gs.group),
   1206:             put(outh, #outh{}),
   1207:             put(sc, SC),
   1208:             DispatchResult = case SC#sconf.dispatch_mod of
   1209:                                  undefined ->


   # AT THIS POINT WE REALIZE THAT WE WOULD LIKE TO ENTER A NEW MODULE...
   (<0.648.0>)> n
   MODULE: yaws_server.erl
   1232:                     ?TC([{record, SC, sconf}]),
   1233:                     ?Debug("Headers = ~s~n", [?format_record(H, headers)]),
   1234:                     ?Debug("Request = ~s~n",
   1235:                            [?format_record(Req, http_request)]),
   1236:                     run_trace_filter(GS, IP, Req, H),
   1237>                     yaws_stats:hit(),
   1238:                     check_keepalive_maxuses(GS, Num),
   1239:                     Call = case yaws_shaper:check(SC, IP) of
   1240:                                allow ->
   1241:                                    call_method(Req#http_request.method,CliSock,
   1242:                                                IPPort,Req,H);

   # ...SO WE LOAD THE 'yaws_stats' MODULE INTO THE DEBUGGER...
   (<0.648.0>)> i yaws_stats
   Interpreted modules: [yaws_server,yaws_stats]

   # ...WE CAN NOW ENTER THAT MODULE
   (<0.648.0>)> s
   MODULE: yaws_stats.erl
   41: stop(Pid) ->
   42:     gen_server:cast(Pid, {stop}).
   43:
   44:
   45: hit() ->
   46>     case get_stats() of
   47:         undefined ->
   48:             ok;
   49:         Pid ->
   50:             gen_server:cast(Pid, {hit})
   51:     end.

   # WE CAN EVALUATE EXPRESSIONS
   (<0.648.0>)> e io:format("Test of eval: ~p~n",[[X*2 || X <- [1,2,3]]]).
   Test of eval: [2,4,6]
   EVALUATED VALUE:
   ok

   # WE CAN LIST BREAK POINTS
   (<0.648.0>)> br

   BREAKPOINTS
    yaws_server:1189  Status=active Trigger=enable Cond=null


   # WE CAN LIST DEBUGGED PROCESSES
   (<0.648.0>)> p
   <0.536.0>  yaws_server:gserv_loop/4   waiting
   <0.648.0>  yaws_server:acceptor0/2    break       yaws_server:1238
   <0.537.0>  yaws_server:POST/4         exit        Reason: shutdown
   <0.516.0>  yaws_server:safe_decode_p  idle
   <0.663.0>  yaws_server:acceptor0/2    running

   # WE CAN LIST ANY MODULE (not just debugged ones)
   (<0.648.0>)> l yaws_api 343
   MODULE: yaws_api.erl
   338:     parse_arg_value(Line, Key, [C|Value], Quote, true).
   339:
   340:
   341: %%
   342:
   343> make_parse_line_reply(Key, Value, Rest) ->
   344:     {{yaws:funreverse(Key, fun yaws:to_lowerchar/1),
   345:       lists:reverse(Value)}, Rest}.
   346:
   347:
   348: -record(mp_parse_state, {

   # TO SEE WHERE WE ARE AT THE MOMENT:
   (<0.648.0>)> a
   MODULE: yaws_server.erl
   1233:                     ?Debug("Headers = ~s~n", [?format_record(H, headers)]),
   1234:                     ?Debug("Request = ~s~n",
   1235:                            [?format_record(Req, http_request)]),
   1236:                     run_trace_filter(GS, IP, Req, H),
   1237:                     yaws_stats:hit(),
   1238>                     check_keepalive_maxuses(GS, Num),
   1239:                     Call = case yaws_shaper:check(SC, IP) of
   1240:                                allow ->
   1241:                                    call_method(Req#http_request.method,CliSock,
   1242:                                                IPPort,Req,H);
   1243:                                {deny, Status, Msg} ->

   # WE CAN CHANGE THE AMOUNT OF CONTEXT TO BE SHOWN
  (<0.648.0>)> x 15

  (<0.648.0>)> a
   MODULE: yaws_server.erl
   1223:                         true  -> {ok, Num+1};
   1224:                         false -> aloop(CliSock, IPPort, GS, Num+1)
   1225:                     end;
   1226:                 closed ->
   1227:                     %% Dispatcher closed the socket
   1228:                     erase_transients(),
   1229:                     {ok, Num+1};
   1230:                 continue ->
   1231:                     ?Debug("SC: ~s", [?format_record(SC, sconf)]),
   1232:                     ?TC([{record, SC, sconf}]),
   1233:                     ?Debug("Headers = ~s~n", [?format_record(H, headers)]),
   1234:                     ?Debug("Request = ~s~n",
   1235:                            [?format_record(Req, http_request)]),
   1236:                     run_trace_filter(GS, IP, Req, H),
   1237:                     yaws_stats:hit(),
   1238>                     check_keepalive_maxuses(GS, Num),
   1239:                     Call = case yaws_shaper:check(SC, IP) of
   1240:                                allow ->
   1241:                                    call_method(Req#http_request.method,CliSock,
   1242:                                                IPPort,Req,H);
   1243:                                {deny, Status, Msg} ->
   1244:                                    deliver_xxx(CliSock, Req, Status, Msg)
   1245:                            end,
   1246:                     Call2 = fix_keepalive_maxuses(Call),
   1247:                     handle_method_result(Call2, CliSock, IPPort,
   1248:                                          GS, Req, H, Num)
   1249:             end;
   1250:         closed ->
   1251:             case yaws_trace:get_type(GS#gs.gconf) of
   1252:                 undefined -> ok;
   1253:                 _         -> yaws_trace:write(from_client, "closed\n")
```

Here is an example of an interactively defined conditional break point
(not possible from the standard GUI debugger):

```erlang
  # FIRST WE DEFINE THE TRIGGER FUNCTION.
  # IT WILL GET THE CURRENT VARIABLE BINDINGS AS THE ARGUMENT
  # AND MUST RETURN EITHER 'true' OR 'false'
  1> F = fun(Bindings) -> case int:get_binding('SSL', Bindings) of
                             {value,nossl} -> true;
                             _ -> false
                          end
         end.
  #Fun<erl_eval.6.50752066>

  # NOW WE SET OUR INTERCTIVELY DEFINED BREAK POINT
  2> edbg:it(yaws_server, 1191, F).
  ok

  # WE RUN OUR TEST...

  # LIST THE DEBUGGED PROCESSES
  3> edbg:plist().
  <0.537.0>  yaws_server:gserv_loop/4   waiting
  <0.538.0>  yaws_server:POST/4         exit        Reason: shutdown
  <0.517.0>  yaws_server:safe_decode_p  idle
  <0.666.0>  yaws_server:acceptor0/2    break       yaws_server:1191
  <0.519.0>  yaws_server:handle_call/3  idle
  <0.718.0>  yaws_server:acceptor0/2    running
  ok

  # ATTACH OURSELVES TO THE FIRST PROCESS STOPPED AT A BREAK POINT
  4> edbg:a().

   (h)elp (a)t <Ctx> (n)ext (s)tep (f)inish (c)ontinue s(k)ip
   (m)essages (t)oggle trace (q)uit (br)eakpoints (p)rocesses
   (de)lete/(di)sable/(en)able/(te)st/(b)reak <Line | Mod Line>
   (v)ar <variable> (e)val <expr> (i)nterpret <Mod>  conte(x)t <Ctx>
   (u)p (d)own (l)ist module <Mod Line <Ctx>>
   MODULE: yaws_server.erl
     1186:     end,
     1187:
     1188:     process_flag(trap_exit, false),
     1189:     init_db(),
     1190:     SSL = GS#gs.ssl,
     1191>     Head = yaws:http_get_headers(CliSock, SSL),
     1192:     process_flag(trap_exit, true),
     1193:     ?Debug("Head = ~p~n", [Head]),
     1194:     case Head of
     1195:         {error, {too_many_headers, ReqTooMany}} ->
     1196:             %% RFC 6585 status code 431

  (<0.666.0>)> v SSL
     'SSL' = nossl
```

<a name="trace-examples"></a>
## TRACE EXAMPLES
```erlang
   # WE WANT TO START TRACING THE FIRST TIME WE ENTER THE 'yaws_server'
   # MODULE, FOR ANY PROCESS. WE WILL ALSO TRACE CALLS TO SOME OTHER
   # Yaws MODULES
   1> edbg:tstart(yaws_server,[yaws,yaws_config,yaws_log]).
   started

   # NOW RUN THE TEST...

   # WHEN THE TEST IS FINISHED WE LIST THE TRACE OUTPUT:
   2> edbg:tlist().

    (h)elp (a)t [<N>] (d)own (u)p (t)op (b)ottom
    (s)how <N> [<ArgN>] (r)etval <N> ra(w) <N>
    (p)agesize <N> (f)ind <M>:<Fx> | <RetVal> (q)uit
      0: <0.536.0> yaws_server:gserv_loop/4
      1:  <0.537.0> yaws_server:peername/2
      3:  <0.536.0> yaws_server:close_accepted_if_max/2
      5:  <0.536.0> yaws_server:acceptor/1
      7:  <0.536.0> yaws_server:gserv_loop/4
      8:   <0.648.0> yaws_server:acceptor0/2
      9:    <0.648.0> yaws_server:do_accept/1
     10:     <0.537.0> yaws_server:aloop/4
     11:      <0.537.0> yaws_server:init_db/0
     13:      <0.537.0> yaws:http_get_headers/2
     14:       <0.537.0> yaws:do_http_get_headers/2
     15:        <0.537.0> yaws:http_recv_request/2
     16:         <0.537.0> yaws:setopts/3
     18:         <0.537.0> yaws:do_recv/3

   # TO INSPECT A PARTICULAR CALL (without drowning in output),
   # WE USE THE (s)how COMMAND WHICH WILL DISPLAY THE FUNCTION
   # CLAUSE HEADS IN ORDER TO HELP US DECIDE WHICH ARGUMENT TO INSPECT.
   tlist> s 1

   Call: yaws_server:peername/2
   -----------------------------------

   peername(CliSock, ssl) ->

   peername(CliSock, nossl) ->

   -----------------------------------

   # TO INSPECT THE SECOND ARGUMENT OF A CALL:
   tlist> s 1 2

   Call: yaws_server:peername/2 , argument 2:
   -----------------------------------
   nossl

   # TO INSPECT THE RETURN VALUE
   tlist> r 1

   Call: yaws_server:peername/2 , return value:
   -----------------------------------
   {{127,0,0,1},56123}

   # IF WE KNOW THE ARGUMENT TO BE A RECORD, WE CAN PRETTY PRINT IT
   tlist> s 173

   Call: yaws_server:handle_normal_request/5
   -----------------------------------

   handle_normal_request(CliSock, ARG, UT = #urltype{type=error}, _, N) ->

   handle_normal_request(CliSock, ARG, UT, Authdirs, N) ->

   -----------------------------------

   tlist> pr 173 2

   Call: yaws_server:handle_normal_request/5 , argument 2:
   -----------------------------------
   #arg{clisock = #Port<0.10952>,
        client_ip_port = {{127,0,0,1},59557},
        headers = #headers{connection = undefined,accept = "*/*",
                           host = "localhost:8080",
                           if_modified_since = undefined,
                           if_match = undefined,if_none_match = undefined,
      ....


   # HERE IS AN EXAMPLE OF HOW WE CAN SEARCH FOR PARTICULAR
   # FUNCTION CALL THAT WE ARE INTERESTED IN.
   # NOTE THAT IT IS ENOUGH TO JUST SPECIFY A PREFIX OF THE FUNCTION NAME
   tlist> f yaws:decode_b
    32:           <0.537.0> yaws:decode_base64/1
    33:            <0.537.0> yaws:decode_base64/2
    34:             <0.537.0> yaws:d/1
      ...


   # WE CAN ALSO SEARCH IN A PARTICULAR ARGUMENT
   # (the 2:nd argument should contain 'packet_size')
   tlist> f yaws:setopts 2 packet_size
    22:         <0.537.0> yaws:setopts/3
    24:         <0.537.0> yaws:do_recv/3
    26:         <0.537.0> yaws:http_collect_headers/5
    ...

   tlist> s 22 2

   Call: yaws:setopts/3 , argument 2:
   -----------------------------------
   [{packet,httph},{packet_size,16384}]


   # WE CAN ALSO SEARCH AMONG THE RETURN VALUES:
   tlist> f GET
   184:           <0.537.0> yaws:make_allow_header/1
   187:          <0.537.0> yaws_server:deliver_accumulated/1
   188:           <0.537.0> yaws:outh_get_content_encoding/0
   190:           <0.537.0> yaws:outh_set_content_encoding/1
     ...

   tlist> r 184

   Call: yaws:make_allow_header/1 , return value:
   -----------------------------------
   ["Allow: GET, POST, OPTIONS, HEAD\r\n"]
 ```