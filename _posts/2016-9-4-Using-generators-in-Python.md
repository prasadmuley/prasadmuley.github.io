---
layout: post
title: Using generators in Python
excerpt: "The pythonic way when you're too much result"
tags: [python, programming, beginner, list, code, MapReduce, MongoDB]
comments: true
language: python
image:
    feature: #asdf
    credit: #asdf
    credtlink: #asdf
---

I love python's generators [generator expression](https://wiki.python.org/moin/Generators). I will explain why I love it.
You must use it whenever you've to return list of items. I learned some interesting stuff about generators.


## What is generators in Python
* Generators are a special class of functions that simplify the task of writing [iterators](https://en.wikipedia.org/wiki/Iterator). 
* Any function containing *yield* keyword or *generator expression* is a generator function.

Lol. It's a so typical and theortical definition of generators.

> Examples
{% highlight python %}

def yeild_keyword(number):
    ''' using yield keyword'''
    for num in range(number):
        yield num


def generator_expression(number):
    '''using gen expression'''
    return (num for num in range(number))
{% endhighlight %}

Here, 
     when function *yield_keyword* gets invoked returns a generator object. I know you're thinking that there is not a single return statement and *how it returns value?*

The yield  keyword or generator expression is detected by Python's bytecode compiler which compiles the function specially as result.


## Difference betweeen generators and regular function
* Regular functions compute a value and return it, but generators return an iterator that returns a stream of values.
* Generators working memory doesn't include all the inputs and outputs. So generators expression is faster than list comprehension when you're computing on large data.

I guess theory is enough now. We're going to use [cProfile](https://docs.python.org/2/library/profile.html#module-cProfile) to analyze time and [memory_profiler](https://pypi.python.org/pypi/memory_profiler).

## Using cProfile

> Example: Calculate square of numbers
{% highlight python %}
from cProfile import run

def square_by_list_comprehensions(list_items):
    """return square of each number
       using list comprehensions"""
    return [num*num for num in list_items]


if __name__ == '__main__':
    run('square_by_list_comprehensions(range(1,100000))')
{% endhighlight %}

> Output:

{% highlight bash %}
$ python squares_by_list_comprehension.py 

         4 function calls in 0.008 seconds

   Ordered by: standard name

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.001    0.001    0.008    0.008 <string>:1(<module>)
        1    0.005    0.005    0.005    0.005 list_comprehension_cprofile.py:3(square_by_list_comprehensions)
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
        1    0.002    0.002    0.002    0.002 {range}

{% endhighlight %}

> Example: Calculate square of numbers using generator

{% highlight python %}
from cProfile import run

def square_by_generator(list_items):
    """return square of each number
       using generator expression"""
    # you can use generator expression
	# return (num for num in list_items)

    for num in list_items:
        yield num * num

if __name__ == '__main__':
    run('square_by_generator(range(1,100000))')

{% endhighlight %}

>Output:

{% highlight bash %}

$ python square_using_generators.py

         4 function calls in 0.002 seconds

   Ordered by: standard name

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.001    0.001    0.002    0.002 <string>:1(<module>)
        1    0.000    0.000    0.000    0.000 yield_cprofile.py:3(square_by_generator)
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
        1    0.002    0.002    0.002    0.002 {range}


{% endhighlight %}

Here,
     generator function takes 0.002 seconds and list comprehension function takes 0.008 seconds.

##  Using memory_profiler

{% highlight python %}
from memory_profiler import profile

@profile
def square_by_list_comprehensions(list_items):
    """return square of each number
       using list comprehensions"""
    return [num*num for num in list_items]


if __name__ == '__main__':
    square_by_list_comprehensions(range(1,10000))

{% endhighlight %}

> Output:

{% highlight bash %}

$ python square_by_list_comprehesion.py 
/usr/local/lib/python2.7/dist-packages/memory_profiler.py:88: UserWarning: psutil module not found. memory_profiler will be slow
  warnings.warn("psutil module not found. memory_profiler will be slow")
Filename: square_by_list_comprehension.py

Line #    Mem usage    Increment   Line Contents
================================================
     3     13.9 MiB      0.0 MiB   @profile
     4                             def square_by_list_comprehensions(list_items):
     5                                 """return square of each number
     6                                    using list comprehensions"""
     7     14.3 MiB      0.4 MiB       return [num*num for num in list_items]


{% endhighlight %}

> Using generators

{% highlight python %}
from memory_profiler import profile

@profile
def square_by_generator(list_items):
    """return square of each number
       using generator expression"""
    for num in list_items:
        yield num * num
    # return (num * num for num in list_items)

if __name__ == '__main__':
    square_by_generator(range(1,10000))

{% endhighlight %}

> Output:

{% highlight bash %}
$ python square_of_numbers_using_generators.py
{% endhighlight %}

Here, 
     generators function didn't use memory but regular function takes around 14MB memory in order to calculate square of numbers.
	 

## Generators vs Mongo's MapReduce
[MapReduce](https://docs.mongodb.com/manual/core/map-reduce/) is being used to improve performance.
Recently, A response from *top_users* function got broken although it has map reduce function.
This function didn't fetch record for last six month (40k records).
So I decided to play with generators and check performance.

> Using MapReduce and cProfile

{% highlight python %}
from db.models import *
import logging
from utils import enable_console_logging
from datetime import datetime
from dateutil.relativedelta import relativedelta
from cProfile import runctx


log = logging.getLogger()
enable_console_logging(log)

def get_top_users(store_id, from_date, to_date, fetch_all=False):

    data = {'more': False, 'top_users': [], 'total': 0}
    points_status_list = [APPROVED, AUTO_APPROVED, REDEEMED,
                          DEDUCTED, EXPIRED, EXHAUSTED]
    top_users_map_f = '''
    function(){
    val = {
           awarded_points : (this.transaction_type == 'award' ? this.points:0),
           redeemed_points : (this.transaction_type == 'redeem' ? this.points:0),
	       first_name: this.user_first_name,
	       last_name: this.user_last_name,
	       user_id: this.user_id
	};
	if(this.transaction_type == 'deduct'){
	    val.awarded_points = 0-val.awarded_points;
        }
        emit(this.user_id, val);
    }
    '''

    top_users_red_f = '''
    function(key, vals){
	red_val = {
	    user_id: key,
	    awarded_points: 0,
            redeemed_points: 0,
	    first_name: null,
	    last_name: null	
	}		
	vals.forEach(function(v){
	    red_val.awarded_points += v.awarded_points;
            red_val.redeemed_points += v.redeemed_points;
	    if(!red_val.first_name)
		red_val.first_name = v.first_name;
	    if(!red_val.last_name)
		red_val.last_name = v.last_name;
	});
	return red_val;
    }
    '''

    result_list = []
    
    for result in Transaction.objects(approved_time__gte = from_date, approved_time__lt = to_date + relativedelta(days = 1),
	                                  store_id = store_id, points_status__in = points_status_list).map_reduce(top_users_map_f,top_users_red_f, 'inline' ):
	result_list.append(result.value)
                
    result_list_sorted = sorted(result_list, key= lambda k: k.get('awarded_points',0), reverse = True) 
    data['total'] = len(result_list_sorted)
    print "Total records: %s" % data['total']
    data['more'] = True if len(result_list_sorted) > start_index + count else False
    if not fetch_all:
	result_list_sorted = result_list_sorted[start_index:start_index+count]

    data['top_users'] = result_list_sorted
    return data

def main():
    connect(db='db_name', host='localhost')
    store_id = 'c99ad71fcd6929dc9161b9b01781ce3a'
    start_date = datetime(1970,1,1)
    end_date = datetime(2016,9,4)
    start_time = datetime.utcnow()
    runctx('get_top_users(store_id, start_date, end_date)',
           globals(), locals())
    end_time = datetime.utcnow()

{% endhighlight %}

> Output:

{% highlight bash %}
$ python top_users_using_mapreduce.py 
Total records: 38619
   
   236111 function calls (236074 primitive calls) in 35.868 seconds

   Ordered by: standard name

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.015    0.015   35.868   35.868 <string>:1(<module>)
    38621    0.004    0.000    0.004    0.000 base.py:1157(_collection)
        1    0.000    0.000    0.000    0.000 base.py:1212(_query)
        2    0.000    0.000    0.001    0.001 base.py:1465(_sub_js_fields)
        3    0.000    0.000    0.000    0.000 base.py:45(__init__)
        2    0.000    0.000    0.000    0.000 base.py:525(clone)
        2    0.000    0.000    0.000    0.000 base.py:533(clone_into)
        1    0.000    0.000    0.000    0.000 base.py:78(__call__)
    38620    0.044    0.000   35.782    0.001 base.py:846(map_reduce)
        1    0.000    0.000    0.000    0.000 calendar.py:110(weekday)
        1    0.000    0.000    0.000    0.000 calendar.py:116(monthrange)
        2    0.000    0.000    0.000    0.000 code.py:44(__new__)
        2    0.000    0.000    0.000    0.000 code.py:65(scope)
        2    0.000    0.000    0.000    0.000 code.py:71(__repr__)
        9    0.000    0.000    0.005    0.001 collection.py:1084(ensure_index)
        1    0.000    0.000   35.721   35.721 collection.py:1598(inline_map_reduce)
       10    0.000    0.000    0.000    0.000 collection.py:162(full_name)
        9    0.000    0.000    0.000    0.000 collection.py:173(name)
       70    0.000    0.000    0.000    0.000 collection.py:182(database)
        9    0.000    0.000    0.000    0.000 collection.py:41(_gen_index_name)
       11    0.000    0.000    0.000    0.000 collection.py:51(__init__)
       10    0.000    0.000    0.000    0.000 collection.py:728(find)
        9    0.000    0.000    0.004    0.000 collection.py:951(create_index)
       12    0.000    0.000    0.000    0.000 common.py:178(validate_positive_float)
       12    0.000    0.000    0.000    0.000 common.py:205(validate_positive_float_or_zero)
       12    0.000    0.000    0.000    0.000 common.py:214(validate_read_preference)
       12    0.000    0.000    0.000    0.000 common.py:227(validate_tag_sets)
       12    0.000    0.000    0.000    0.000 common.py:271(validate_uuid_subtype)
       12    0.000    0.000    0.000    0.000 common.py:375(__init__)
       12    0.000    0.000    0.000    0.000 common.py:396(__init__)
        5    0.000    0.000    0.000    0.000 common.py:4(_import_class)
       12    0.000    0.000    0.000    0.000 common.py:438(__set_options)
       12    0.000    0.000    0.000    0.000 common.py:474(__get_write_concern)
       24    0.000    0.000    0.000    0.000 common.py:536(__get_slave_okay)
       25    0.000    0.000    0.000    0.000 common.py:554(__get_read_pref)
       23    0.000    0.000    0.000    0.000 common.py:570(__get_acceptable_latency)
       23    0.000    0.000    0.000    0.000 common.py:596(__get_tag_sets)
       13    0.000    0.000    0.000    0.000 common.py:618(__get_uuid_subtype)
       12    0.000    0.000    0.000    0.000 common.py:633(__get_safe)
       24    0.000    0.000    0.000    0.000 common.py:73(validate_boolean)
        1    0.000    0.000    0.000    0.000 connection.py:128(get_db)
        1    0.000    0.000    0.000    0.000 connection.py:82(get_connection)
       30    0.000    0.000    0.000    0.000 copy.py:101(_copy_immutable)
        6    0.000    0.000    0.000    0.000 copy.py:113(_copy_with_constructor)
        4    0.000    0.000    0.000    0.000 copy.py:306(_reconstruct)
       40    0.000    0.000    0.000    0.000 copy.py:66(copy)
        4    0.000    0.000    0.000    0.000 copy_reg.py:92(__newobj__)
        2    0.000    0.000    0.000    0.000 copy_reg.py:95(_slotnames)
       20    0.000    0.000   35.723    1.786 cursor.py:1000(_refresh)
       10    0.000    0.000    0.000    0.000 cursor.py:1082(__iter__)
       20    0.000    0.000   35.723    1.786 cursor.py:1085(next)
       10    0.000    0.000    0.000    0.000 cursor.py:200(conn_id)
       10    0.000    0.000    0.000    0.000 cursor.py:216(__del__)
       10    0.000    0.000    0.000    0.000 cursor.py:295(__query_spec)
       10    0.000    0.000    0.000    0.000 cursor.py:381(__query_options)
       10    0.000    0.000    0.000    0.000 cursor.py:391(__check_okay_to_chain)
       10    0.000    0.000    0.000    0.000 cursor.py:438(limit)
       10    0.000    0.000    0.000    0.000 cursor.py:76(__init__)
       10    0.000    0.000   35.722    3.572 cursor.py:912(__send_message)
       78    0.000    0.000    0.000    0.000 database.py:112(connection)
       39    0.000    0.000    0.000    0.000 database.py:121(name)
       11    0.000    0.000    0.000    0.000 database.py:183(__getattr__)
       11    0.000    0.000    0.000    0.000 database.py:193(__getitem__)
       10    0.000    0.000    0.000    0.000 database.py:264(_fix_outgoing)
       10    0.000    0.000   35.725    3.572 database.py:277(_command)
       10    0.000    0.000   35.725    3.572 database.py:349(command)
        1    0.000    0.000    0.000    0.000 database.py:39(__init__)
        1    0.000    0.000    0.000    0.000 document.py:139(_get_db)
      2/1    0.000    0.000    0.005    0.005 document.py:144(_get_collection)
        9    0.000    0.000    0.000    0.000 document.py:25(includes_cls)
        1    0.000    0.000    0.000    0.000 document.py:533(_get_collection_name)
        1    0.000    0.000    0.005    0.005 document.py:540(ensure_indexes)
    38619    0.012    0.000    0.012    0.000 document.py:738(__init__)
        4    0.000    0.000    0.000    0.000 document.py:762(_lookup_field)
        3    0.000    0.000    0.000    0.000 field_list.py:10(__init__)
        2    0.000    0.000    0.000    0.000 fields.py:377(to_mongo)
        2    0.000    0.000    0.000    0.000 fields.py:421(prepare_query_value)
        7    0.000    0.000    0.000    0.000 fields.py:87(prepare_query_value)
       10    0.000    0.000    0.000    0.000 helpers.py:126(_check_command_response)
        1    0.000    0.000    0.000    0.000 helpers.py:230(_check_database_name)
       18    0.000    0.000    0.000    0.000 helpers.py:37(_index_list)
        9    0.000    0.000    0.000    0.000 helpers.py:53(_index_document)
       10    0.004    0.000    0.118    0.012 helpers.py:80(_unpack_response)
        1    0.000    0.000    0.005    0.005 manager.py:27(__get__)
       10    0.000    0.000    0.000    0.000 member.py:148(get_socket)
       10    0.000    0.000    0.000    0.000 member.py:155(maybe_return_socket)
       10    0.000    0.000    0.000    0.000 mongo_client.py:1078(__check_bson_size)
       20    0.004    0.000   35.603    1.780 mongo_client.py:1146(__receive_data_on_socket)
       10    0.000    0.000   35.603    3.560 mongo_client.py:1161(__receive_message_on_socket)
       10    0.000    0.000   35.603    3.560 mongo_client.py:1177(__send_and_receive)
       10    0.000    0.000   35.604    3.560 mongo_client.py:1190(_send_message_with_response)
        1    0.000    0.000    0.000    0.000 mongo_client.py:1299(__getattr__)
        1    0.000    0.000    0.000    0.000 mongo_client.py:1310(__getitem__)
        9    0.000    0.000    0.000    0.000 mongo_client.py:397(_cached)
        9    0.000    0.000    0.000    0.000 mongo_client.py:407(_cache_index)
       10    0.000    0.000    0.000    0.000 mongo_client.py:497(__check_auth)
       30    0.000    0.000    0.000    0.000 mongo_client.py:515(__member_property)
       20    0.000    0.000    0.000    0.000 mongo_client.py:556(is_mongos)
       10    0.000    0.000    0.000    0.000 mongo_client.py:603(auto_start_request)
       10    0.000    0.000    0.000    0.000 mongo_client.py:608(get_document_class)
       10    0.000    0.000    0.000    0.000 mongo_client.py:621(tz_aware)
       10    0.000    0.000    0.000    0.000 mongo_client.py:629(max_bson_size)
       10    0.000    0.000    0.000    0.000 mongo_client.py:778(__ensure_member)
       10    0.000    0.000    0.000    0.000 mongo_client.py:909(__socket)
       10    0.000    0.000    0.000    0.000 pool.py:100(__ne__)
       10    0.000    0.000    0.000    0.000 pool.py:103(__hash__)
       10    0.000    0.000    0.000    0.000 pool.py:309(get_socket)
       10    0.000    0.000    0.000    0.000 pool.py:415(maybe_return_socket)
       10    0.000    0.000    0.000    0.000 pool.py:436(_return_socket)
       10    0.000    0.000    0.000    0.000 pool.py:456(_check)
       20    0.000    0.000    0.000    0.000 pool.py:539(_get_request_state)
       10    0.000    0.000    0.000    0.000 pool.py:81(set_wire_version_range)
       30    0.000    0.000    0.000    0.000 pool.py:95(__eq__)
        4    0.000    0.000    0.001    0.000 re.py:144(sub)
        4    0.000    0.000    0.001    0.000 re.py:226(_compile)
        1    0.000    0.000    0.000    0.000 relativedelta.py:109(__init__)
        1    0.000    0.000    0.000    0.000 relativedelta.py:202(_fix)
        1    0.000    0.000    0.000    0.000 relativedelta.py:245(__radd__)
       10    0.000    0.000    0.000    0.000 socket.py:223(meth)
       48    0.000    0.000    0.000    0.000 son.py:102(__setitem__)
       11    0.000    0.000    0.000    0.000 son.py:111(keys)
       67    0.000    0.000    0.000    0.000 son.py:122(__iter__)
       48    0.000    0.000    0.000    0.000 son.py:180(update)
        1    0.000    0.000    0.000    0.000 son.py:196(get)
        1    0.000    0.000    0.000    0.000 son.py:213(__len__)
       19    0.000    0.000    0.000    0.000 son.py:85(__init__)
       19    0.000    0.000    0.000    0.000 son.py:91(__new__)
    19/10    0.000    0.000    0.000    0.000 son.py:96(__repr__)
        8    0.000    0.000    0.000    0.000 sre_compile.py:178(_compile_charset)
        8    0.000    0.000    0.000    0.000 sre_compile.py:207(_optimize_charset)
       22    0.000    0.000    0.000    0.000 sre_compile.py:24(_identityfunction)
        2    0.000    0.000    0.000    0.000 sre_compile.py:258(_mk_bitmap)
     10/2    0.000    0.000    0.000    0.000 sre_compile.py:32(_compile)
        6    0.000    0.000    0.000    0.000 sre_compile.py:354(_simple)
        2    0.000    0.000    0.000    0.000 sre_compile.py:361(_compile_info)
        4    0.000    0.000    0.000    0.000 sre_compile.py:474(isstring)
        2    0.000    0.000    0.001    0.000 sre_compile.py:480(_code)
        2    0.000    0.000    0.001    0.001 sre_compile.py:495(compile)
       24    0.000    0.000    0.000    0.000 sre_parse.py:126(__len__)
       42    0.000    0.000    0.000    0.000 sre_parse.py:130(__getitem__)
        6    0.000    0.000    0.000    0.000 sre_parse.py:134(__setitem__)
       18    0.000    0.000    0.000    0.000 sre_parse.py:138(append)
     16/8    0.000    0.000    0.000    0.000 sre_parse.py:140(getwidth)
        2    0.000    0.000    0.000    0.000 sre_parse.py:178(__init__)
       62    0.000    0.000    0.000    0.000 sre_parse.py:182(__next)
       32    0.000    0.000    0.000    0.000 sre_parse.py:195(match)
       50    0.000    0.000    0.000    0.000 sre_parse.py:201(get)
       10    0.000    0.000    0.000    0.000 sre_parse.py:257(_escape)
      4/2    0.000    0.000    0.000    0.000 sre_parse.py:301(_parse_sub)
      4/2    0.000    0.000    0.000    0.000 sre_parse.py:379(_parse)
        2    0.000    0.000    0.000    0.000 sre_parse.py:663(parse)
        2    0.000    0.000    0.000    0.000 sre_parse.py:67(__init__)
        2    0.000    0.000    0.000    0.000 sre_parse.py:72(opengroup)
        2    0.000    0.000    0.000    0.000 sre_parse.py:83(closegroup)
       10    0.000    0.000    0.000    0.000 sre_parse.py:90(__init__)
    38619    0.008    0.000    0.020    0.000 top_users_using_mapreduce.py:102(<lambda>)
        1    0.025    0.025   35.853   35.853 top_users_using_mapreduce.py:53(get_top_users)
       20    0.000    0.000    0.000    0.000 thread_util.py:106(get)
       10    0.000    0.000    0.000    0.000 thread_util.py:230(acquire)
       10    0.000    0.000    0.000    0.000 thread_util.py:255(release)
       10    0.000    0.000    0.000    0.000 thread_util.py:275(release)
       20    0.000    0.000    0.000    0.000 thread_util.py:49(acquire)
       20    0.000    0.000    0.000    0.000 thread_util.py:52(release)
       20    0.000    0.000    0.000    0.000 thread_util.py:92(_make_vigil)
       10    0.000    0.000    0.000    0.000 threading.py:225(_is_owned)
       10    0.000    0.000    0.000    0.000 threading.py:276(notify)
       10    0.000    0.000    0.000    0.000 threading.py:62(_note)
        1    0.000    0.000    0.000    0.000 transform.py:31(query)
        1    0.000    0.000    0.000    0.000 visitor.py:116(__and__)
        4    0.000    0.000    0.000    0.000 visitor.py:153(__init__)
        2    0.000    0.000    0.000    0.000 visitor.py:156(accept)
        2    0.000    0.000    0.000    0.000 visitor.py:159(empty)
        1    0.000    0.000    0.000    0.000 visitor.py:20(visit_query)
        1    0.000    0.000    0.000    0.000 visitor.py:70(__init__)
        1    0.000    0.000    0.000    0.000 visitor.py:79(visit_query)
        1    0.000    0.000    0.000    0.000 visitor.py:90(to_query)
        1    0.000    0.000    0.000    0.000 visitor.py:98(_combine)
        2    0.000    0.000    0.000    0.000 {_sre.compile}
       70    0.000    0.000    0.000    0.000 {_struct.unpack}
        5    0.000    0.000    0.000    0.000 {abs}
       10    0.114    0.011    0.114    0.011 {bson._cbson.decode_all}
       25    0.000    0.000    0.000    0.000 {built-in method __new__ of type object at 0x890140}
       18    0.000    0.000    0.000    0.000 {built-in method utcnow}
      114    0.000    0.000    0.000    0.000 {getattr}
       88    0.000    0.000    0.000    0.000 {hasattr}
       10    0.000    0.000    0.000    0.000 {hash}
       24    0.000    0.000    0.000    0.000 {id}
      407    0.000    0.000    0.000    0.000 {isinstance}
  462/455    0.000    0.000    0.000    0.000 {len}
        4    0.000    0.000    0.000    0.000 {method '__reduce_ex__' of 'object' objects}
       60    0.000    0.000    0.000    0.000 {method 'acquire' of 'thread.lock' objects}
       10    0.000    0.000    0.000    0.000 {method 'add' of 'set' objects}
    38904    0.003    0.000    0.003    0.000 {method 'append' of 'list' objects}
       18    0.000    0.000    0.000    0.000 {method 'copy' of 'dict' objects}
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
        6    0.000    0.000    0.000    0.000 {method 'extend' of 'list' objects}
    38789    0.012    0.000    0.012    0.000 {method 'get' of 'dict' objects}
        2    0.000    0.000    0.000    0.000 {method 'get' of 'dictproxy' objects}
       18    0.000    0.000    0.000    0.000 {method 'isdigit' of 'str' objects}
        4    0.000    0.000    0.000    0.000 {method 'items' of 'dict' objects}
       41    0.000    0.000    0.000    0.000 {method 'iteritems' of 'dict' objects}
       23    0.000    0.000    0.000    0.000 {method 'join' of 'str' objects}
        9    0.000    0.000    0.000    0.000 {method 'join' of 'unicode' objects}
       10    0.000    0.000    0.000    0.000 {method 'lower' of 'str' objects}
        6    0.000    0.000    0.000    0.000 {method 'lstrip' of 'str' objects}
       78    0.000    0.000    0.000    0.000 {method 'pop' of 'dict' objects}
        3    0.000    0.000    0.000    0.000 {method 'pop' of 'list' objects}
       10    0.000    0.000    0.000    0.000 {method 'pop' of 'set' objects}
       10    0.000    0.000    0.000    0.000 {method 'popleft' of 'collections.deque' objects}
       96   35.599    0.371   35.599    0.371 {method 'recv' of '_socket.socket' objects}
       50    0.000    0.000    0.000    0.000 {method 'release' of 'thread.lock' objects}
        2    0.000    0.000    0.000    0.000 {method 'remove' of 'list' objects}
        1    0.000    0.000    0.000    0.000 {method 'replace' of 'datetime.datetime' objects}
       10    0.000    0.000    0.000    0.000 {method 'replace' of 'str' objects}
       10    0.000    0.000    0.000    0.000 {method 'sendall' of '_socket.socket' objects}
        4    0.000    0.000    0.000    0.000 {method 'split' of 'str' objects}
       20    0.000    0.000    0.000    0.000 {method 'startswith' of 'str' objects}
        4    0.000    0.000    0.000    0.000 {method 'sub' of '_sre.SRE_Pattern' objects}
       38    0.000    0.000    0.000    0.000 {method 'update' of 'dict' objects}
       12    0.000    0.000    0.000    0.000 {method 'values' of 'dict' objects}
        1    0.000    0.000    0.000    0.000 {method 'weekday' of 'datetime.date' objects}
       21    0.000    0.000    0.000    0.000 {min}
       26    0.000    0.000    0.000    0.000 {ord}
       20    0.000    0.000    0.000    0.000 {posix.getpid}
       10    0.000    0.000    0.000    0.000 {pymongo._cmessage._query_message}
        6    0.000    0.000    0.000    0.000 {range}
       10    0.000    0.000    0.000    0.000 {repr}
       40    0.000    0.000    0.000    0.000 {setattr}
        2    0.017    0.008    0.037    0.019 {sorted}
       20    0.000    0.000    0.000    0.000 {time.time}


(env) 

{% endhighlight %}

* So top_user function took 36 seconds. Let's see generator example:

> Using generators and cProfile
{% highlight python %}
from db.models import *
import logging
from utils import enable_console_logging
from datetime import datetime
from dateutil.relativedelta import relativedelta
from cProfile import runctx


def get_transactions_using_generator(**kwargs):
    for lt in Transaction.objects(**kwargs):
        yield lt


def get_top_users_generators(merchant_id, from_date, to_date, fetch_all=False):

    data = {'more': False, 'top_users': [], 'total': 0}
    points_status_list = [APPROVED, AUTO_APPROVED, REDEEMED,
                          DEDUCTED, EXPIRED, EXHAUSTED]
    result_dict = dict()

    kwargs = dict(store_id = store_id, points_status__in = points_status_list,
                  approved_time__lt = to_date + relativedelta(days=1),
                  approved_time__gte = from_date)
    
    for result in get_transactions_using_generator(**kwargs):
        if result.transaction_type == 'award':
            if result.user_email in result_dict:
                result_dict[result.user_email] += result.points 
            else:
                result_dict[result.user_email] = result.points
        else:
            if result.user_email in result_dict:
                result_dict[result.user_email] -= result.points
            else:
                result_dict[result.user_email] = (-result.points)
                
    result_list_sorted = [{user: result_dict[user]} for user in
                          sorted(result_dict, key=result_dict.get, reverse = True)]
    data['total'] = len(result_list_sorted)
    print "Total records: %s" % data['total']
    data['more'] = True if len(result_list_sorted) > start_index + count else False
    if not fetch_all:
	result_dict_sorted = result_list_sorted[start_index:start_index+count]

    data['top_users'] = result_list_sorted
    return data


def main():
    connect(db='ss_db', host='localhost')
    # get merchant ids who have loyalty enabled
    store_id = 'c99ad71fcd6929dc9161b9b01781ce3a'
    start_date = datetime(1970,1,1)
    end_date = datetime(2016,9,4)
    start_time = datetime.utcnow()
    runctx('get_top_users_using_generators(merchant_id, start_date, end_date)',
           globals(), locals())
    end_time = datetime.utcnow()


if __name__ == '__main__':
    main()

{% endhighlight %}

> Output using generators

{% highlight bash %}

$ python top_users_using_generator.py 
Total records: 38643
         32932865 function calls (32788303 primitive calls) in 28.247 seconds

   Ordered by: standard name

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.014    0.014   28.247   28.247 <string>:1(<module>)
    84206    0.339    0.000   27.415    0.000 base.py:1131(next)
        1    0.000    0.000    0.000    0.000 base.py:1157(_collection)
        1    0.000    0.000    0.000    0.000 base.py:1164(_cursor_args)
    84206    0.027    0.000    0.027    0.000 base.py:1178(_cursor)
        1    0.000    0.000    0.000    0.000 base.py:1212(_query)
        2    0.000    0.000    0.000    0.000 base.py:45(__init__)
        1    0.000    0.000    0.000    0.000 base.py:525(clone)
        1    0.000    0.000    0.000    0.000 base.py:533(clone_into)
        1    0.000    0.000    0.000    0.000 base.py:78(__call__)
        1    0.000    0.000    0.000    0.000 calendar.py:110(weekday)
        1    0.000    0.000    0.000    0.000 calendar.py:116(monthrange)
        9    0.000    0.000    0.005    0.001 collection.py:1084(ensure_index)
       23    0.000    0.000    0.000    0.000 collection.py:162(full_name)
        9    0.000    0.000    0.000    0.000 collection.py:173(name)
    84287    0.019    0.000    0.019    0.000 collection.py:182(database)
        9    0.000    0.000    0.000    0.000 collection.py:41(_gen_index_name)
       10    0.000    0.000    0.000    0.000 collection.py:51(__init__)
       10    0.000    0.000    0.000    0.000 collection.py:728(find)
        9    0.000    0.000    0.004    0.000 collection.py:951(create_index)
       11    0.000    0.000    0.000    0.000 common.py:178(validate_positive_float)
       11    0.000    0.000    0.000    0.000 common.py:205(validate_positive_float_or_zero)
       11    0.000    0.000    0.000    0.000 common.py:214(validate_read_preference)
       11    0.000    0.000    0.000    0.000 common.py:227(validate_tag_sets)
       11    0.000    0.000    0.000    0.000 common.py:271(validate_uuid_subtype)
       11    0.000    0.000    0.000    0.000 common.py:375(__init__)
       11    0.000    0.000    0.000    0.000 common.py:396(__init__)
  2923321    0.947    0.000    1.204    0.000 common.py:4(_import_class)
       11    0.000    0.000    0.000    0.000 common.py:438(__set_options)
       11    0.000    0.000    0.000    0.000 common.py:474(__get_write_concern)
       20    0.000    0.000    0.000    0.000 common.py:536(__get_slave_okay)
       22    0.000    0.000    0.000    0.000 common.py:554(__get_read_pref)
       21    0.000    0.000    0.000    0.000 common.py:570(__get_acceptable_latency)
       21    0.000    0.000    0.000    0.000 common.py:596(__get_tag_sets)
       12    0.000    0.000    0.000    0.000 common.py:618(__get_uuid_subtype)
       11    0.000    0.000    0.000    0.000 common.py:633(__get_safe)
       22    0.000    0.000    0.000    0.000 common.py:73(validate_boolean)
        1    0.000    0.000    0.000    0.000 connection.py:128(get_db)
        1    0.000    0.000    0.000    0.000 connection.py:82(get_connection)
       15    0.000    0.000    0.000    0.000 copy.py:101(_copy_immutable)
        3    0.000    0.000    0.000    0.000 copy.py:113(_copy_with_constructor)
        2    0.000    0.000    0.000    0.000 copy.py:306(_reconstruct)
       20    0.000    0.000    0.000    0.000 copy.py:66(copy)
        2    0.000    0.000    0.000    0.000 copy_reg.py:92(__newobj__)
        2    0.000    0.000    0.000    0.000 copy_reg.py:95(_slotnames)
       33    0.001    0.000    2.606    0.079 cursor.py:1000(_refresh)
        9    0.000    0.000    0.000    0.000 cursor.py:1082(__iter__)
    84224    0.251    0.000    3.005    0.000 cursor.py:1085(next)
        9    0.000    0.000    0.000    0.000 cursor.py:200(conn_id)
       10    0.000    0.000    0.000    0.000 cursor.py:216(__del__)
       10    0.000    0.000    0.000    0.000 cursor.py:295(__query_spec)
       10    0.000    0.000    0.000    0.000 cursor.py:381(__query_options)
        9    0.000    0.000    0.000    0.000 cursor.py:391(__check_okay_to_chain)
        9    0.000    0.000    0.000    0.000 cursor.py:438(limit)
       10    0.000    0.000    0.000    0.000 cursor.py:76(__init__)
       23    0.003    0.000    2.605    0.113 cursor.py:912(__send_message)
       90    0.000    0.000    0.000    0.000 database.py:112(connection)
       37    0.000    0.000    0.000    0.000 database.py:121(name)
       10    0.000    0.000    0.000    0.000 database.py:183(__getattr__)
       10    0.000    0.000    0.000    0.000 database.py:193(__getitem__)
    84214    0.110    0.000    0.110    0.000 database.py:264(_fix_outgoing)
        9    0.000    0.000    0.004    0.000 database.py:277(_command)
        9    0.000    0.000    0.004    0.000 database.py:349(command)
        1    0.000    0.000    0.000    0.000 database.py:39(__init__)
  3031380    4.777    0.000    8.645    0.000 document.py:113(__setattr__)
        1    0.000    0.000    0.000    0.000 document.py:139(_get_db)
      2/1    0.000    0.000    0.005    0.005 document.py:144(_get_collection)
        9    0.000    0.000    0.000    0.000 document.py:25(includes_cls)
    84205    3.628    0.000   15.997    0.000 document.py:35(__init__)
        1    0.000    0.000    0.000    0.000 document.py:533(_get_collection_name)
    84205    4.207    0.000   24.046    0.000 document.py:539(_from_son)
        1    0.000    0.000    0.005    0.005 document.py:540(ensure_indexes)
  1804294    1.200    0.000    1.200    0.000 document.py:547(<genexpr>)
        4    0.000    0.000    0.000    0.000 document.py:762(_lookup_field)
    84205    0.329    0.000    0.435    0.000 document.py:824(__set_field_display)
        2    0.000    0.000    0.000    0.000 field_list.py:10(__init__)
        1    0.000    0.000    0.000    0.000 field_list.py:67(__nonzero__)
   252697    0.031    0.000    0.031    0.000 fields.py:126(to_python)
   336669    0.112    0.000    0.112    0.000 fields.py:173(to_python)
228757/84205    1.167    0.000    1.802    0.000 fields.py:233(to_python)
        4    0.000    0.000    0.000    0.000 fields.py:241(to_python)
   168272    0.057    0.000    0.057    0.000 fields.py:346(to_python)
        2    0.000    0.000    0.000    0.000 fields.py:377(to_mongo)
    84205    0.042    0.000    0.050    0.000 fields.py:390(to_python)
        2    0.000    0.000    0.000    0.000 fields.py:421(prepare_query_value)
   794037    0.261    0.000    0.342    0.000 fields.py:62(to_python)
  1227086    0.465    0.000    0.600    0.000 fields.py:84(__get__)
        7    0.000    0.000    0.000    0.000 fields.py:87(prepare_query_value)
  2610355    2.194    0.000    3.868    0.000 fields.py:94(__set__)
        9    0.000    0.000    0.000    0.000 helpers.py:126(_check_command_response)
        1    0.000    0.000    0.000    0.000 helpers.py:230(_check_database_name)
       18    0.000    0.000    0.000    0.000 helpers.py:37(_index_list)
        9    0.000    0.000    0.000    0.000 helpers.py:53(_index_document)
       23    0.013    0.001    1.120    0.049 helpers.py:80(_unpack_response)
        1    0.000    0.000    0.005    0.005 manager.py:27(__get__)
       23    0.000    0.000    0.001    0.000 member.py:148(get_socket)
       23    0.000    0.000    0.001    0.000 member.py:155(maybe_return_socket)
       23    0.000    0.000    0.000    0.000 mongo_client.py:1078(__check_bson_size)
       46    0.030    0.001    1.478    0.032 mongo_client.py:1146(__receive_data_on_socket)
       23    0.000    0.000    1.479    0.064 mongo_client.py:1161(__receive_message_on_socket)
       23    0.000    0.000    1.480    0.064 mongo_client.py:1177(__send_and_receive)
       23    0.000    0.000    1.482    0.064 mongo_client.py:1190(_send_message_with_response)
        1    0.000    0.000    0.000    0.000 mongo_client.py:1299(__getattr__)
        1    0.000    0.000    0.000    0.000 mongo_client.py:1310(__getitem__)
        9    0.000    0.000    0.000    0.000 mongo_client.py:397(_cached)
        9    0.000    0.000    0.000    0.000 mongo_client.py:407(_cache_index)
       23    0.000    0.000    0.000    0.000 mongo_client.py:497(__check_auth)
       30    0.000    0.000    0.000    0.000 mongo_client.py:515(__member_property)
       20    0.000    0.000    0.000    0.000 mongo_client.py:556(is_mongos)
       23    0.000    0.000    0.000    0.000 mongo_client.py:603(auto_start_request)
       10    0.000    0.000    0.000    0.000 mongo_client.py:608(get_document_class)
       10    0.000    0.000    0.000    0.000 mongo_client.py:621(tz_aware)
       10    0.000    0.000    0.000    0.000 mongo_client.py:629(max_bson_size)
       23    0.000    0.000    0.000    0.000 mongo_client.py:778(__ensure_member)
       23    0.000    0.000    0.001    0.000 mongo_client.py:909(__socket)
    84205    0.068    0.000    0.104    0.000 objectid.py:203(__validate)
    84205    0.025    0.000    0.129    0.000 objectid.py:77(__init__)
       23    0.000    0.000    0.000    0.000 pool.py:100(__ne__)
       23    0.000    0.000    0.000    0.000 pool.py:103(__hash__)
       23    0.000    0.000    0.001    0.000 pool.py:309(get_socket)
       23    0.000    0.000    0.001    0.000 pool.py:415(maybe_return_socket)
       23    0.000    0.000    0.001    0.000 pool.py:436(_return_socket)
       12    0.000    0.000    0.000    0.000 pool.py:44(_closed)
       23    0.000    0.000    0.000    0.000 pool.py:456(_check)
       46    0.000    0.000    0.000    0.000 pool.py:539(_get_request_state)
       23    0.000    0.000    0.000    0.000 pool.py:81(set_wire_version_range)
       69    0.000    0.000    0.000    0.000 pool.py:95(__eq__)
        1    0.000    0.000    0.000    0.000 queryset.py:25(__iter__)
    84206    0.023    0.000   27.661    0.000 queryset.py:65(_iter_results)
      843    0.208    0.000   27.638    0.033 queryset.py:83(_populate_cache)
        1    0.000    0.000    0.000    0.000 relativedelta.py:109(__init__)
        1    0.000    0.000    0.000    0.000 relativedelta.py:202(_fix)
        1    0.000    0.000    0.000    0.000 relativedelta.py:245(__radd__)
   168410    0.023    0.000    0.023    0.000 signals.py:30(<lambda>)
       35    0.000    0.000    0.000    0.000 socket.py:223(meth)
       43    0.000    0.000    0.000    0.000 son.py:102(__setitem__)
        9    0.000    0.000    0.000    0.000 son.py:111(keys)
       61    0.000    0.000    0.000    0.000 son.py:122(__iter__)
   168455    0.103    0.000    0.135    0.000 son.py:180(update)
    84223    0.137    0.000    0.272    0.000 son.py:85(__init__)
    84223    0.106    0.000    0.135    0.000 son.py:91(__new__)
     18/9    0.000    0.000    0.000    0.000 son.py:96(__repr__)
    84206    0.218    0.000   27.884    0.000 top_users_using_generator.py:11(get_transactions_using_generator)
        1    0.185    0.185   28.233   28.233 top_users_using_generator.py:16(get_top_users_using_generators)
       46    0.000    0.000    0.000    0.000 thread_util.py:106(get)
       23    0.000    0.000    0.000    0.000 thread_util.py:230(acquire)
       23    0.000    0.000    0.000    0.000 thread_util.py:255(release)
       23    0.000    0.000    0.000    0.000 thread_util.py:275(release)
       46    0.000    0.000    0.000    0.000 thread_util.py:49(acquire)
       46    0.000    0.000    0.000    0.000 thread_util.py:52(release)
       46    0.000    0.000    0.000    0.000 thread_util.py:92(_make_vigil)
       23    0.000    0.000    0.000    0.000 threading.py:225(_is_owned)
       23    0.000    0.000    0.000    0.000 threading.py:276(notify)
       23    0.000    0.000    0.000    0.000 threading.py:62(_note)
        1    0.000    0.000    0.000    0.000 transform.py:31(query)
        1    0.000    0.000    0.000    0.000 visitor.py:116(__and__)
        3    0.000    0.000    0.000    0.000 visitor.py:153(__init__)
        2    0.000    0.000    0.000    0.000 visitor.py:156(accept)
        2    0.000    0.000    0.000    0.000 visitor.py:159(empty)
        1    0.000    0.000    0.000    0.000 visitor.py:20(visit_query)
        1    0.000    0.000    0.000    0.000 visitor.py:70(__init__)
        1    0.000    0.000    0.000    0.000 visitor.py:79(visit_query)
        1    0.000    0.000    0.000    0.000 visitor.py:90(to_query)
        1    0.000    0.000    0.000    0.000 visitor.py:98(_combine)
      161    0.000    0.000    0.000    0.000 {_struct.unpack}
        5    0.000    0.000    0.000    0.000 {abs}
       23    0.977    0.042    1.107    0.048 {bson._cbson.decode_all}
    84225    0.028    0.000    0.028    0.000 {built-in method __new__ of type object at 0x890140}
       18    0.000    0.000    0.000    0.000 {built-in method utcnow}
      289    0.000    0.000    0.000    0.000 {callable}
   890406    0.396    0.000    0.853    0.000 {getattr}
   505701    0.247    0.000    0.247    0.000 {hasattr}
       23    0.000    0.000    0.000    0.000 {hash}
       48    0.000    0.000    0.000    0.000 {id}
  4078717    0.833    0.000    0.833    0.000 {isinstance}
   170389    0.015    0.000    0.015    0.000 {len}
        2    0.000    0.000    0.000    0.000 {method '__reduce_ex__' of 'object' objects}
      138    0.000    0.000    0.000    0.000 {method 'acquire' of 'thread.lock' objects}
       23    0.000    0.000    0.000    0.000 {method 'add' of 'set' objects}
    84303    0.015    0.000    0.015    0.000 {method 'append' of 'list' objects}
       18    0.000    0.000    0.000    0.000 {method 'copy' of 'dict' objects}
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
       12    0.000    0.000    0.000    0.000 {method 'fileno' of '_socket.socket' objects}
  8565209    1.190    0.000    1.190    0.000 {method 'get' of 'dict' objects}
        2    0.000    0.000    0.000    0.000 {method 'get' of 'dictproxy' objects}
       18    0.000    0.000    0.000    0.000 {method 'isdigit' of 'str' objects}
   288755    0.184    0.000    0.184    0.000 {method 'items' of 'dict' objects}
   421063    0.059    0.000    0.059    0.000 {method 'iteritems' of 'dict' objects}
       22    0.000    0.000    0.000    0.000 {method 'join' of 'str' objects}
        9    0.000    0.000    0.000    0.000 {method 'join' of 'unicode' objects}
        9    0.000    0.000    0.000    0.000 {method 'lower' of 'str' objects}
        6    0.000    0.000    0.000    0.000 {method 'lstrip' of 'str' objects}
    84277    0.016    0.000    0.016    0.000 {method 'pop' of 'dict' objects}
        3    0.000    0.000    0.000    0.000 {method 'pop' of 'list' objects}
       23    0.000    0.000    0.000    0.000 {method 'pop' of 'set' objects}
    84214    0.010    0.000    0.010    0.000 {method 'popleft' of 'collections.deque' objects}
       76    1.448    0.019    1.448    0.019 {method 'recv' of '_socket.socket' objects}
      115    0.000    0.000    0.000    0.000 {method 'release' of 'thread.lock' objects}
        1    0.000    0.000    0.000    0.000 {method 'replace' of 'datetime.datetime' objects}
        9    0.000    0.000    0.000    0.000 {method 'replace' of 'str' objects}
       23    0.000    0.000    0.000    0.000 {method 'sendall' of '_socket.socket' objects}
        4    0.000    0.000    0.000    0.000 {method 'split' of 'str' objects}
       18    0.000    0.000    0.000    0.000 {method 'startswith' of 'str' objects}
       45    0.000    0.000    0.000    0.000 {method 'update' of 'dict' objects}
       11    0.000    0.000    0.000    0.000 {method 'values' of 'dict' objects}
        1    0.000    0.000    0.000    0.000 {method 'weekday' of 'datetime.date' objects}
        1    0.000    0.000    0.000    0.000 {min}
       46    0.000    0.000    0.000    0.000 {posix.getpid}
       13    0.000    0.000    0.000    0.000 {pymongo._cmessage._get_more_message}
       10    0.000    0.000    0.000    0.000 {pymongo._cmessage._query_message}
        9    0.000    0.000    0.000    0.000 {repr}
       12    0.000    0.000    0.000    0.000 {select.select}
  2610375    1.350    0.000    8.808    0.000 {setattr}
    84207    0.151    0.000    0.151    0.000 {sorted}
       46    0.000    0.000    0.000    0.000 {time.time}


(env)
{% endhighlight %}

Here,
   MapReduce takes more time than generators. Response time will improve if I use generators instead of map reduce.

## When to use generators
* It gives you lazy evaluation. Actually instead of returning the all elements , it returns them one by one.
* Useful for calculating large sets of results.
  * if we don't know whether we need all results or not , in that case we don't need to allocate the memory for all results at the same time.
