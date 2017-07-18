## What is the Osmanthus?
Osmanthus is a framework for rules engines, a lightweight library and based on MVEL library. It is more convenient than drools, developers define rules & flows with XML file, it supplies below useful XML node:
### Rule node

* rule: a single rule

```xml
<rule id="rulefirst" name="rulefirst" priority="0" exclusive="true" multipleTimes="5" valid="true">
    <condition><![CDATA[true]]></condition>
    <action><![CDATA[System.out.println("this is a rule,hah")]]></action>
</rule>
```   
* ruleset: set of rules, it extends the rule, so that means it has all attributes that rule has.

```xml
<ruleset name="set" id="set">
    <rule id="rulefirst" name="rulefirst" priority="0" exclusive="true" multipleTimes="5" valid="true">
        <condition><![CDATA[true]]></condition>
        <action><![CDATA[System.out.println("this is the first rule,hah")]]></action>
    </rule>
    <rule id="rulesecond" name="rulesecond" priority="0" exclusive="true" multipleTimes="5" valid="true">
        <condition><![CDATA[true]]></condition>
        <action><![CDATA[System.out.println("this is the second rule,hah")]]></action>
    </rule>
</ruleset>
```

### Flow Control node

* flow, whole flow control node, it can contain the rule,ruleset,split,paraller and merge node.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<flow id="singleflow1">
    <start id="start" toNodeId="feerule"/>
    <ruleset id="feerule" fromNodeId="start" toNodeId="split1" external="true"/>
    <split id="split1" fromNodeId="feerule">
        <constraint toNodeId="card">
            <condition><![CDATA[fee>1]]></condition>
        </constraint>
        <constraint toNodeId="end">
            <condition><![CDATA[fee<=1]]></condition>
        </constraint>
    </split>
    <ruleset id="card" fromNodeId="split1" toNodeId="p1" external="true"/>
    <rule id="p1" fromNodeId="card" toNodeId="p2">
        <condition><![CDATA[1==1]]></condition>
        <action><![CDATA[rule1="p1"]]></action>
    </rule>
    <rule id="p2" fromNodeId="p1" toNodeId="p3">
        <condition><![CDATA[2==2]]></condition>
        <action><![CDATA[rule2="p2"]]></action>
    </rule>
    <rule id="p3" fromNodeId="p2" toNodeId="p4">
        <condition><![CDATA[1==1]]></condition>
        <action><![CDATA[rule3="p3"]]></action>
    </rule>
    <rule id="p4" fromNodeId="p3" toNodeId="p5">
        <condition><![CDATA[2==2]]></condition>
        <action><![CDATA[rule4="p4"]]></action>
    </rule>
    <rule id="p5" fromNodeId="p4" toNodeId="end">
        <condition><![CDATA[2==2]]></condition>
        <action><![CDATA[rule5="p5"]]></action>
    </rule>
    <end id="end"/>
</flow>
```

* split, one flow control node, also extends rule node, has more than one "constraint", but only one "constraint"'s condition is true.

```xml
<split id="split1" >
    <constraint toNodeId="card">
        <condition><![CDATA[fee>1]]></condition>
    </constraint>
    <constraint toNodeId="end">
        <condition><![CDATA[fee<=1]]></condition>
    </constraint>
</split>
```

* parallel, also a flow control node, it controls the execution of rules with parallel model. if your defined flow XML file contains this sort of node, only run this flow with MultipleThreadExecutor Java class. one "line" node that means the system will create a new thread to execute its next nodes.
"sync" attribute means the main thread have to wait, until all its sub thread run over.
"timeout", unit is second. e.g, timeout=100, that means the mail thread will wait 100s, at this time, if there is any sub thread is running, will throw timeout exception.

```xml
<parallel id="multiple" sync="true" timeout="100">
    <line toNodeId="p1"/>
    <line toNodeId="p2"/>
</parallel>
```

* merge, a merge node, that means this node can merge the execution results of multiple thread, the merge node don't any thing, it equals "End" node.

```xml
<merge id="merge" fromNodeId="p3,p4" toNodeId="p5"/>
```

## Features

 * Lightweight library and easy to learn API
 * Development with xml programming model
 * All conditions & Actions supports MVEL expression languages
 * Useful abstractions to define business rules and apply them easily with Java
 * The ability to create composite rules from primitive xml codes.
 * Parallel mode to execute multiple rules.
 * Support decision tree and score cards rules.
 

## Hello world-Score Card Rules

### Rules in XML file:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ruleset name="card" id="card">
 	<rule id="gc1">
		<condition><![CDATA[fee<=1]]></condition>
		<action><![CDATA[weight=weight-100]]></action>
	</rule>
	<rule id="gc2">
		<condition><![CDATA[isBlackName]]></condition>
		<action><![CDATA[weight=weight-200]]></action>
	</rule>
 	<rule id="gc3">
		<condition><![CDATA[name=='test']]></condition>
		<action><![CDATA[weight=weight-500]]></action>
	</rule>
	<rule id="gc4">
		<condition><![CDATA[reg ~= "[1-9][0-9]{4,}"]]></condition>
		<action><![CDATA[weight=weight-600]]></action>
	</rule>
</ruleset>
```

## The rules of Decision Tree
 In order to implments the decision tree data structure of rules, we have to use the "SPlIT" node to help us to create more than one branch, but here we call the banrach as "constraint", that contatins one condition, if its condition is true, it will go to "toNodeId" node.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<flow id="flow1">
    <start id="start" toNodeId="feerule"/>
    <ruleset id="feerule" fromNodeId="start" toNodeId="split1" external="true"/>
    <split id="split1" fromNodeId="feerule">
        <constraint toNodeId="card">
            <condition><![CDATA[fee>1]]></condition>
        </constraint>
        <constraint toNodeId="end">
            <condition><![CDATA[fee<=1]]></condition>
        </constraint>
    </split>
    <ruleset id="card" fromNodeId="split1" toNodeId="p5" external="true"/>
    <rule id="p5" fromNodeId="merge" toNodeId="end">
        <condition><![CDATA[2==2]]></condition>
        <action><![CDATA[rule5="p5"]]></action>
    </rule>
    <end id="end"/>
</flow>
```

## Execute rules with Multiple Thread
If you want to run rules with multiple thead model, you have to use the "Parallel" node to help us to create more than one line, xml code as below：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<flow id="multiflow">
    <start id="start" toRuleId="feerule"/>
    <ruleset id="feerule" fromRuleId="start" toRuleId="split1" external="true"/>
    <split id="split1" fromRuleId="feerule">
        <constraint toRuleId="card">
            <condition><![CDATA[fee>1]]></condition>
        </constraint>
        <constraint toRuleId="end">
            <condition><![CDATA[fee<=1]]></condition>
        </constraint>
    </split>
    <ruleset id="card" fromRuleId="split1" toRuleId="mlines" external="true"/>
    <parallel id="mlines" fromRuleId="card" sync="true" timeout="100" toRuleId="merge">
        <line toRuleId="p1"/>
        <line toRuleId="p2"/>
    </parallel>
    <rule id="p1" fromRuleId="mlines" toRuleId="end">
        <condition><![CDATA[1==1]]></condition>
        <action><![CDATA[rule1="p1"]]></action>
    </rule>
    <rule id="p2" fromRuleId="mlines" toRuleId="mlines2">
        <condition><![CDATA[2==2]]></condition>
        <action><![CDATA[rule2="p2"]]></action>
    </rule>
    <parallel id="mlines2" fromRuleId="p2" sync="true" timeout="100" toRuleId="merge">
        <line toRuleId="p3"/>
        <line toRuleId="p4"/>
    </parallel>
    <rule id="p3" fromRuleId="mlines2" toRuleId="merge">
        <condition><![CDATA[1==1]]></condition>
        <action><![CDATA[rule3="p3"]]></action>
    </rule>
    <rule id="p4" fromRuleId="mlines2" toRuleId="merge">
        <condition><![CDATA[2==2]]></condition>
        <action><![CDATA[rule4="p4"]]></action>
    </rule>
    <merge id="merge" fromRuleId="p3,p4" lineCnt="2" toRuleId="p5"/>
    <rule id="p5" fromRuleId="merge" toRuleId="end">
        <condition><![CDATA[2==2]]></condition>
        <action><![CDATA[rule5="p5"]]></action>
    </rule>
    <end id="end"/>
</flow>

```

### JAVA Unit Test Code
```Java
package com.github.wei86609.osmanthus;

import java.util.concurrent.Executors;

import junit.framework.TestCase;

import com.github.wei86609.osmanthus.event.DefaultEventListener;
import com.github.wei86609.osmanthus.event.Event;
import com.github.wei86609.osmanthus.event.ParallelEventListener;
import com.github.wei86609.osmanthus.rule.executor.EndNodeExecutor;
import com.github.wei86609.osmanthus.rule.executor.MergeNodeExecutor;
import com.github.wei86609.osmanthus.rule.executor.MvelRuleExecutor;
import com.github.wei86609.osmanthus.rule.executor.ParallelRuleExecutor;
import com.github.wei86609.osmanthus.rule.executor.SplitRuleExecutor;
import com.github.wei86609.osmanthus.rule.executor.StartNodeExecutor;
import com.github.wei86609.osmanthus.rule.executor.ruleset.GeneralRuleSetExecutor;
import com.github.wei86609.osmanthus.rule.handler.GeneralRuleSetHandler;
import com.github.wei86609.osmanthus.rule.intercepter.DefaultIntercepter;

public class FlowEngineTest extends TestCase {

   protected FlowEngine buildMultiThreadFlowEngine() {
        final FlowEngine engine = new FlowEngine();
        //RuleExceutors
        ParallelRuleExecutor parallelRuleExecutor=new ParallelRuleExecutor();
        MvelRuleExecutor ruleExecutor = new MvelRuleExecutor();
        engine.addRuleExecutor(new SplitRuleExecutor());
        engine.addRuleExecutor(new StartNodeExecutor());
        engine.addRuleExecutor(new MergeNodeExecutor());
        engine.addRuleExecutor(new EndNodeExecutor());
        engine.addRuleExecutor(parallelRuleExecutor);
        engine.addRuleExecutor(ruleExecutor);
        //RuleSetExcecutors
        GeneralRuleSetExecutor generalRuleSetExecutor = new GeneralRuleSetExecutor();
        generalRuleSetExecutor.setRuleExecutor(ruleExecutor);
        generalRuleSetExecutor.addRuleSetHandler(new GeneralRuleSetHandler());
        engine.addRuleExecutor(generalRuleSetExecutor);
        //add event listener & thread pool for executer of parallel rule
        parallelRuleExecutor.setParallelEventListener(new ParallelEventListener(){
            public void startNewEvent(Event event, String ruleId) throws Exception{
                event.setCurrentRuleId(ruleId);
                engine.execute(event);
            }
        });
        parallelRuleExecutor.setThreadPool(Executors.newCachedThreadPool());
        return engine;
    }
    protected FlowEngine buildSingleThreadFlowEngine() {
        FlowEngine engine = new FlowEngine();
        //add event listener and intercepter
        engine.addEventListener(new DefaultEventListener());
        engine.addRuleInterceptor(new DefaultIntercepter());
        // add RuleExceutors
        MvelRuleExecutor ruleExecutor = new MvelRuleExecutor();
        engine.addRuleExecutor(new SplitRuleExecutor());
        engine.addRuleExecutor(new StartNodeExecutor());
        engine.addRuleExecutor(new EndNodeExecutor());
        engine.addRuleExecutor(ruleExecutor);
        // add RuleSetExcecutors
        GeneralRuleSetExecutor generalRuleSetExecutor = new GeneralRuleSetExecutor();
        generalRuleSetExecutor.setRuleExecutor(ruleExecutor);
        generalRuleSetExecutor.addRuleSetHandler(new GeneralRuleSetHandler());
        engine.addRuleExecutor(generalRuleSetExecutor);
        return engine;
    }
    public void testSingleThreadExecute() {
        Event event=new Event();
        event.setFlowId("singleflow1");
        event.setEventId("eventid");
        event.addVar("salary", 5000);
        event.addVar("weight", 500);
        event.addVar("isBlackName", true);
        event.addVar("fee", 500);
        event.addVar("name", "test");
        event.addVar("reg", "12312");
        event.addVar("idNum", "350823198809122917");
        try {
            buildSingleThreadFlowEngine().execute(event);
            System.out.println(""+event);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    public void testMultiThreadExecute() {
        Event event=new Event();
        event.setFlowId("multiflow");
        event.setEventId("muleventid");
        event.addVar("salary", 5000);
        event.addVar("weight", 500);
        event.addVar("isBlackName", true);
        event.addVar("fee", 500);
        event.addVar("name", "test");
        event.addVar("reg", "12312");
        event.addVar("idNum", "350823198809122917");
        try {
            buildMultiThreadFlowEngine().execute(event);
            System.out.println(""+event);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

```
## Example of "Guess Number"
 The "Guess Number" is a classical example to help us to understand the rule engine, so here can also implement the example by Osmanthus.
 
 ```xml
 <?xml version="1.0" encoding="UTF-8"?>
 <rule id="guessnumber">
     <condition><![CDATA[true]]></condition>
     <action><![CDATA[
    import java.io.*;
    import java.util.Random;
    //
    // Seed the random number
    //
    $num = new Random().nextInt(100);
    System.out.println($num);
    $guesses = 0;
    $in = -1;
    //
    // Setup the STDIN line reader.
    //
    $linereader = new BufferedReader(new InputStreamReader(System.in));
    System.out.print("I'm Thinking of a Number Between 1 and 100... Can you guess what it is? ");
    // Main program loop
    //
    while ($in != $num) {
        if ($in != -1) {
            System.out.print("Nope.  The number is: " + ($num < $in ? "Lower" : "Higher") + ".  What's your next guess? ");
        }
        if (($in = $linereader.readLine().trim()) == empty) $in = -2;
            $guesses++;
    }
    System.out.println("You got it!  It took you " + $guesses + " tries");    
]]></action>
 </rule>
 
 ```
### Code example
 ```java

 ```
 
