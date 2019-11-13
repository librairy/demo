# load documents
curl --user "demo:2019" -X POST -H "Content-Type: application/json" -d @json/load-en-documents.json http://localhost:8081/documents

# train a topic model
curl --user "demo:2019" -X POST -H "Content-Type: application/json" -d @json/topic-model-en.json http://localhost:8081/topics

# run the model
docker run -it --rm -p 7777:7777 <repository>:<version>

# inference topics
curl -X POST -H "Content-Type: application/json" -d @json/topic-inference-en.json http://localhost:7777/classes

# annotate docs
curl --user "demo:2019" -X POST -H "Content-Type: application/json" -d @json/annotate-en.json http://localhost:8081/annotations
