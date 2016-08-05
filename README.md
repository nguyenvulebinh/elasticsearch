# Tìm hiểu về Elasticsearch

## Giới thiệu
+ **Tổng quan**

  Elasticsearch là một "open-source search engine" được xây dựng dựa trên  Apache Lucene (full-text search-engine library), một thư viện cung cấp khả năng tìm kiếm trên cơ sở dữ liệu văn bản có hiệu năng cao nhất hiện nay. Tuy nhiên Lucene chỉ là một thư viện, để tích hợp bào ứng dụng cần nhiều bước phức tạp. Elasticsearch sử dụng Lucene và ẩn đi sự phức tạp của nó bằng cách cung cấp các RESTful API.
  
  Elasticsearch không chỉ cung cấp khả năng tìm kiếm full-text, có còn cung cấp các tính năng khác như: 
    - A distributed real-time document store where every field is indexed and searchable
    - A distributed search engine with real-time analytics
    - Capable of scaling to hundreds of servers and petabytes of structured and unstructured data
  
  Tất cả các tính năng này được gói gọn vào một máy chủ độc lập, người sử dụng chỉ cần quan tâm tới việc gọi các RESTful API, việc phân tán dữ liệu ra các server khác nhau là do Elasticsearch quản lý. 
  
  Elasticsearch cung cấp rất nhiều tính năng và cấu hình để thuận tiện cho ngươ sử dụng, đồng thời là một open source cung cấp dưới giấy phép Apache 2 nên có thể tải source về để tùy biến theo ý muốn.
  
+ **Mô tả kiến trúc**

  Đọc các khái niệm cơ bản về cluster, node, index, type, document, Shards & Replicas theo tài liệu [1]
  
  Elasticsearch phân chia một cách logic thành các cluster. Mỗi cluster là một instance của Elasticsearch chứa một tập các node. Các node có cùng cluster name thì thuộc cùng một cluster. Node chính là nơi lưu trữ dữ liệu thực tế, các documnet, index. Trong mỗi một cluster sẽ có một node master đảm nhận nhiệm vụ tạo các node mới hoặc hủy node, chuyển hướng yêu cầu tới các node khác trong cùng cluster. Các node có thể nằm trên cùng một máy hoặc trên nhiều máy tính khác nhau. Khi các node nằm trên các máy khác nhau thì cần cấu hình để liên lạc giữa các node có cùng cluster name.

  Dữ liệu lưu trữ trong các index(có thể hình dung như một thư mục hoặc là các package), trong mỗi package này chứa các type (tưởng tượng như các class), document là các các thể hiện của class này, mỗi document sẽ có một id định danh riêng có thể do hệ thống tự tạo hoặc người dùng định nghĩa. Như vậy một document sẽ được định danh bởi 3 yếu tố {index}/{type}/{id}
  
  Mỗi index sẽ được phân chia thành các mảnh khác nhau được gọi là shard, các shard của một index có thể được lưu trữ tại một hoặc một vài node. Mỗi shard có thể có các bản sao của nó được gọi là các replica, số lương replica của một shard và số lượng shard có thể được cấu hình tùy ý. Nguyên tắc lưu trữ các shard và các replica là "Không có một replica của một shard nào được lưu trữ trên cùng một node". Điều này đảm bảo tăng tính sẵn sàng và chịu lỗi.
  
  Elasticsearch có cơ chế quản lý xung đột các dữ liệu theo nguyên tắc "Khi một document được lấy xuống, trong lúc clientA đang sửa đổi mà document đó được cập nhật bới một clientB nào đó, thì khi clientA cập nhật sẽ thất bại". Đọc thêm phần Dealing with Conflicts tài liệu [4] và Optimistic Concurrency Control tài liệu [5]  
  
  Đọc thêm phần mô tả về Life Inside a Cluster theo tài liệu [2]

## Cài đặt

  Thưc hiện cài đặt Elasticsearch theo tài liệu [3].
  
  Việc giao tiếp với Elasticsearch có thể thực hiện theo 3 cách
  
  - Đơn giản nhất sử dụng tool curl để gửi request trưc tiếp tới server Elasticsearch
  - Sử dụng API do Elastic.co cung cấp, Elastic cung cấp thư viện client cho rất nhiều ngôn ngữ như java, php, javascript,.. Tham khảo theo tài liệu [6]
  - Cách trực quan nhất là sử dụng Kibana Sence để thực hiện giao tiếp với Elasticsearch. Tuy nhiên cách này muốn thực hiện được thì trước đó phải có index rồi. Thưc hiện cài đặt Kibana theo tài liệu [7] và cài đặt thêm Sence theo tài liệu [8] 

## Hướng dẫn sử dụng
  
  + **Cấu trúc request**
  
  Elasticsearch sử dụng RESTful API với JSON thông qua HTTP/HTTPS. Cấu trúc của một request sẽ là (ví dụ sử dụng curl):
  
  ```
  curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
  ```
  Trong đó:

    - `VERB`: The appropriate HTTP _method_ or _verb_: `GET`, `POST`, `PUT`, `HEAD`, or `DELETE`.

    - `PROTOCOL`: Either `http` or `https` (if you have an `https` proxy in front of Elasticsearch.)

    - `HOST`: The hostname of any node in your Elasticsearch cluster, or +localhost+ for a node on your local machine.

    - `PORT`: The port running the Elasticsearch HTTP service, which defaults to `9200`.

    - `PATH`: API Endpoint (for example `_count` will return the number of documents in the cluster). Path may contain multiple components, such as `_cluster/stats` or `_nodes/stats/jvm`

    - `QUERY_STRING`: Any optional query-string parameters (for example `?pretty` will _pretty-print_  the JSON response to make it easier to read.)

    - `BODY`:A JSON-encoded request body (if the request needs one.)
  
  
## Tài liệu tham khảo

[1] https://www.elastic.co/guide/en/elasticsearch/reference/current/_basic_concepts.html

[2] https://www.elastic.co/guide/en/elasticsearch/guide/current/distributed-cluster.html

[3] https://www.elastic.co/guide/en/elasticsearch/guide/current/running-elasticsearch.html

[4] https://www.elastic.co/guide/en/elasticsearch/guide/current/version-control.html

[5] https://www.elastic.co/guide/en/elasticsearch/guide/current/optimistic-concurrency-control.html

[6] https://www.elastic.co/guide/en/elasticsearch/client/index.html

[7] https://www.elastic.co/guide/en/kibana/current/setup.html

[8] https://www.elastic.co/guide/en/sense/current/installing.html
