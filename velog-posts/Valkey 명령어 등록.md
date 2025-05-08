<h1 id="명령어-등록">명령어 등록</h1>
<blockquote>
<p>echo와 동일한 기능을 수행하는 새로운 명령어를 추가한다.</p>
</blockquote>
<h2 id="명령어-등록을-위해-수정해야-하는-파일">명령어 등록을 위해 수정해야 하는 파일</h2>
<ol>
<li><code>server.c</code> : 명령어 실행 로직을 구현하는 실제 함수 정의 파일</li>
<li><code>server.h</code> : 명령어의 함수 선언을 추가하는 헤더 파일</li>
<li><code>commands.def</code> : 모든 명령어의 메타데이터와 실제 실행 함수 포인터를 선언하는 정적 선언 파일<h2 id="명령어-실행-로직-구현">명령어 실행 로직 구현</h2>
<img alt="" src="https://velog.velcdn.com/images/sjhgd107/post/271a538a-2118-4b71-bdad-bd400e5178b8/image.png" /></li>
</ol>
<h2 id="프로토타입에-함수-선언-추가">프로토타입에 함수 선언 추가</h2>
<p><img alt="" src="https://velog.velcdn.com/images/sjhgd107/post/e8874b13-564f-44a1-968a-91757970fc93/image.png" /></p>
<h2 id="명령어-메타데이터-및-함수-포인터-선언">명령어 메타데이터 및 함수 포인터 선언</h2>
<p><img alt="" src="https://velog.velcdn.com/images/sjhgd107/post/4b2b615c-c061-4498-9fa9-d9479790ae83/image.png" /></p>
<pre><code>{MAKE_CMD(&quot;echoJunHyeok&quot;,&quot;Returns the given string.&quot;,&quot;O(1)&quot;,&quot;1.0.0&quot;,CMD_DOC_NONE,NULL,NULL,&quot;connection&quot;,COMMAND_GROUP_CONNECTION,ECHO_History,0,ECHO_Tips,0,echoJunHyeokCommand,2,CMD_LOADING|CMD_STALE|CMD_FAST,ACL_CATEGORY_CONNECTION,ECHO_Keyspecs,0,NULL,1),.args=ECHO_Args},</code></pre><p><code>commands.h</code>의 구조체 선언에 따르면 <code>serverCommandArg</code> 구조는 다음과 같다.
<code>echo</code>와 동일한 기능을 수행하되 명령어의 이름이 달라야하므로 <code>declared_name</code>을 &quot;echoJunHyeok&quot;으로 정의한다.
또한 함수를 새로 만들어줬으므로 함수 포인터를 변경하고자 <code>serverCommandProc</code>를 &quot;echoJunHyeokCommand&quot;로 정의한다.</p>
<pre><code class="language-c">struct serverCommand {
    /* Declarative data */
    const char *declared_name;    /* A string representing the command declared_name.
                                   * It is a const char * for native commands and SDS for module commands. */
    const char *summary;          /* Summary of the command (optional). */
    const char *complexity;       /* Complexity description (optional). */
    const char *since;            /* Debut version of the command (optional). */
    int doc_flags;                /* Flags for documentation (see CMD_DOC_*). */
    const char *replaced_by;      /* In case the command is deprecated, this is the successor command. */
    const char *deprecated_since; /* In case the command is deprecated, when did it happen? */
    serverCommandGroup group;     /* Command group */
    commandHistory *history;      /* History of the command */
    int num_history;
    const char **tips; /* An array of strings that are meant to be tips for clients/proxies regarding this command */
    int num_tips;
    serverCommandProc *proc; /* Command implementation */
    int arity;               /* Number of arguments, it is possible to use -N to say &gt;= N */
    uint64_t flags;          /* Command flags, see CMD_*. */
    uint64_t acl_categories; /* ACl categories, see ACL_CATEGORY_*. */
    keySpec *key_specs;
    int key_specs_num;
    /* Use a function to determine keys arguments in a command line.
     * Used for Cluster redirect (may be NULL) */
    serverGetKeysProc *getkeys_proc;
    int num_args; /* Length of args array. */
    /* Array of subcommands (may be NULL) */
    struct serverCommand *subcommands;
    /* Array of arguments (may be NULL) */
    struct serverCommandArg *args;
#ifdef LOG_REQ_RES
    /* Reply schema */
    struct jsonObject *reply_schema;
#endif

    /* Runtime populated data */
    long long microseconds, calls, rejected_calls, failed_calls;
    int id;           /* Command ID. This is a progressive ID starting from 0 that
                         is assigned at runtime, and is used in order to check
                         ACLs. A connection is able to execute a given command if
                         the user associated to the connection has this command
                         bit set in the bitmap of allowed commands. */
    sds fullname;     /* Includes parent name if any: &quot;parentcmd|childcmd&quot;. Unchanged if command is renamed. */
    sds current_name; /* Same as fullname, becomes a separate string if command is renamed. */
    struct hdr_histogram
        *latency_histogram;        /* Points to the command latency command histogram (unit of time nanosecond). */
    keySpec legacy_range_key_spec; /* The legacy (first,last,step) key spec is
                                    * still maintained (if applicable) so that
                                    * we can still support the reply format of
                                    * COMMAND INFO and COMMAND GETKEYS */
    hashtable *subcommands_ht;     /* Subcommands hash table. The key is the subcommand sds name
                                    * (not the fullname), and the value is the serverCommand structure pointer. */
    struct serverCommand *parent;
    struct ValkeyModuleCommand *module_cmd; /* A pointer to the module command data (NULL if native command) */
};</code></pre>
<h2 id="빌드">빌드</h2>
<p>기존 valkey 빌드를 지우고 새로 빌드하기 위해 <code>make clean &amp;&amp; make</code>를 실행한다.
<img alt="" src="https://velog.velcdn.com/images/sjhgd107/post/30ab8e6a-1a9f-4164-9902-30d40648ed43/image.png" /></p>
<h2 id="실행">실행</h2>
<p><code>valkey-server</code>를 실행하고 새로 등록한 명령어 <code>echoJunHyeok</code>을 실행하여 확인한다.</p>
<p><img alt="" src="https://velog.velcdn.com/images/sjhgd107/post/b1512c0c-3513-4b8f-9688-858a6dd02a8e/image.png" /></p>