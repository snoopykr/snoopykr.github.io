---
layout: single
title: "Python Simple RestAPI"
categories : Python
tag: [Python, Flask, RestAPI]
---

Python Flask을 이용한 Simple RestAPI

### main.py

```python
from flask import Flask, jsonify, request

app = Flask(__name__)

items = [
    {"id": 1, "name": "apple", "price": 1000},
    {"id": 2, "name": "banana", "price": 1500},
    {"id": 3, "name": "orange", "price": 2000}
]

# GET /api/hello
@app.route("/api/hello")
def hello():
    return "Hello, World!"

# GET /api/items
@app.route("/api/items")
def get_items():
    return jsonify(items)

# GET /api/items/{id}
@app.route("/api/items/<int:item_id>")
def get_item(item_id):
    for item in items:
        if item["id"] == item_id:
            return jsonify(item)
    return jsonify({"message": "Item not found"}), 404

# POST /api/items
@app.route("/api/items", methods=["POST"])
def create_item():
    item = request.get_json()
    items.append(item)
    return jsonify(item), 201

# PUT /api/items/{id}
@app.route("/api/items/<int:item_id>", methods=["PUT"])
def update_item(item_id):
    item = request.get_json()
    for i in range(len(items)):
        if items[i]["id"] == item_id:
            items[i] = item
            return jsonify(item)
    return jsonify({"message": "Item not found"}), 404

# DELETE /api/items/{id}
@app.route("/api/items/<int:item_id>", methods=["DELETE"])
def delete_item(item_id):
    for i in range(len(items)):
        if items[i]["id"] == item_id:
            del items[i]
            return jsonify({"message": "Item deleted"}), 200
    return jsonify({"message": "Item not found"}), 404

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8000, debug=True)
```


> 로컬에서 사용할 목적이라면 host을 지정하지 않아도 되지만 도커나 외부에 노출이 되어야 한다면 host="0.0.0.0"를 추가해서 외부에서도 접속이 가능하게 처리되어야 한다. 내무, 외부 상관 없이 필수로 추가하는 것을 추천.


### Rest API 호출

List (Read)
- GET
- http://localhost:8000/api/items


Item (Read)
- GET
- http://localhost:8000/api/items/1


Item 추가 (Create)
- POST
- http://localhost:8000/api/items
- {"id": 4, "name": "test", "price": 2000}

Item 변경 (Update)
- PUT
- http://localhost:8000/api/items/4
- {"id": 4, "name": "test1111", "price": 20001111}

Item 삭제 (Delete)
- DELETE
- http://localhost:8000/api/items/4