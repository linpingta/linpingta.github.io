---
layout: post
title: "为Django博客添加Elasticsearch搜索"
date: 2016-10-04 18:56:56 +0800
comments: true
categories: [Django, Elasticsearch] 
keywords: 
description: 
---

翻翻自己的markdown文件，发现上一篇关于Django的文章已经是大概两年前了。虽然文章没有更新，但由于前一阵开始把博客从Github上迁移回自己写的Django网站，相关工作还是做了一些的，这篇博客算是一篇关于Elasicsearch应用的快手博客。

简介
Elasticsearch是基于Luence开发的实时搜索框架，它提供分布式可扩展的服务，以RESTful API的形式提供对外服务，官网地址在[这里](https://www.elastic.co/products/elasticsearch)。

安装
下载地址在[这里](https://www.elastic.co/downloads/elasticsearch)，安装步骤也可以按官网设置，启动服务后，Elasticsearch默认通过9200端口对外提供服务，我采用的版本是2.4.1

博客添加搜索
这是应用的主要内容，为了达到这一目的，我们要完成以下几件事：
1.将博客内容添加进Elasticsearch索引
2.在博客内添加搜索访问逻辑

假设我们的博客model如下：

	class Blog(models.Model):
	    title = models.CharField(max_length=100)
	    content = models.TextField()
	    pub_date = models.DateField()
	    tags = models.ManyToManyField(Tag)
	    author = models.ForeignKey(Author, on_delete=models.CASCADE)

1. 添加索引
采用Elasticsearch的[Python客户端](https://elasticsearch-py.readthedocs.io/en/master/)，创建index名为linpingta-blog,type为article的访问索引，使用bulk方法批量建立索引：

		from elasticsearch import Elasticsearch
		from elasticsearch import helpers
		from blogs.models import Blog
	
		if __name__ == '__main__':
	        es = Elasticsearch()
	
	        # create index
            blogs = Blog.objects.all()
            actions = []
            count = 0
            for blog in blogs:
                    print blog.id, blog.title.encode("utf-8")
                    action = {
                            "_index": "linpingta-blog",
                            "_type": "article",
                            "_id": blog.id,
                            "_source": {
                                    "title": blog.title,
                                    "content": blog.content,
                                    "author": blog.author.name
                            }
                    }
                    actions.append(action)
                    count = count + 1
                    if count > 500:
                            helpers.bulk(es, actions)
                            actions = []
                            count = 0

            if count > 0:
                    helpers.bulk(es, actions)
索引建立完毕后，可以做简单的本地验证：(

		search_word = "python"
		res = es.search(index="linpingta-blog", body={
                "query": {
                        "match":{
                                "content": search_word
                        }
                }
        })
        print "Got %d Hits" % res["hits"]["total"]
        for hit in res['hits']['hits']:
                print ("%(author)s: %(title)s" % hit["_source"]).encode("utf-8")

输出结果：

	Got 34 Hits
	褚桐: Python修饰器
	褚桐: excel_convertor
	...

2.博客内添加访问逻辑
在相应页面内添加form，在接受form的views方法里调用Elasticsearch，再将搜索到的博客标题对应到博客，返回给展示页面

blog.html：

    <form class="m-blog-search" action="/blogs/search/" method="post">
        {\% csrf_token \%}
        <input id="search_word" type="text" name="search_word" placeholder="博客标题搜索...">
        <input type="submit" value="搜索">
    </form>
    
 views.py
 
	def search(request):
	    if request.method == 'POST':
	        search_word = request.POST['search_word']
	        es = Elasticsearch()
	        res = es.search(index="linpingta-blog", body={
	                "query": {
	                        "match":{
	                                "title": search_word
	                        }
	                }
	        })
	        blogs = []
	        for hit in res['hits']['hits']:
	                title = hit['_source']['title']
	                blog = Blog.objects.get(title=title)
	                blogs.append(blog)
	        context = { 'blogs': blogs }
		        return render_to_response('blogs/search_result.html', context=context)
	    else:
	        return HttpResponseRedirect('/blogs/')

3.其它
Elasticsearch自带的中文分词功能比较简陋，网上一般采用[ik](https://github.com/medcl/elasticsearch-analysis-ik)提供中文搜索的支持。但因为网络原因，配合2.4.1版本的ik一直不能正常下载，考虑时间成本，最后只能暂时先放弃。如文章开头所说，这篇博客只是举了一个Elasticsearch的实例，没有涉及Elasticsearch的原理，这点有待以后有机会再做学习。
