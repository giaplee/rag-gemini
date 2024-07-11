# RAG=Gemini-Sentence-Transformer-Qdrant
This project is a demonstrate how to use a vector database and gemini as chatbot in RAG model
* Please check the code in try it with notebook file named `rag_encoding.ipynb` at the same folder
* The dataset file in dataset folder
* Link tới <a href="https://github.com/giaplee/rag-gemini">`github`</a>
* Link tới <a href="https://github.com/giaplee/rag-gemini/blob/master/rag_encoding.ipynb" >`Jupyter Notebook`</a>
* Chạy trên Colab
  <td>
  <a href="https://colab.research.google.com/github/giaplee/rag-gemini/blob/master/rag_encoding.ipynb" target="_blank">
  <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open in Colab" style="max-width: 100%; height: auto;">
  </a>
  </td>

# RAG với Qdrant và Gemini
---------------------------
### Author: __Giap Le (Harrison)__
Đây là một bài hướng dẫn thực hành cơ bản và nâng cao về việc áp dụng mô hình RAG để tăng độ chính xác của thông tin phàn hồi từ chatbot dựa vào nguồn dữ liệu được kiểm soát và giới hạn ở __vector database__ (ở đây tôi sử dụng `Qdrant`).
Sau đó sử dụng gemini như một chatbot để tương tác với dữ liệu và người dùng.
Sử dụng kỹ thuật prompt engineering là `few-shot prompting` >> nghĩa là hãy đưa vào cho chatbot model nhiều hơn 1 ngữ cảnh (dữ liệu hoăc/và gợi ý)
và yêu cầu chatbot chỉ sử dụng dữ liệu đã cung cấp làm nguồn thông tin để trả lời.

Ở đây chúng ta sử dụng **sentence-transformers** và pre-trained model: **all-MiniML-L6-v2** từ __HuggingFace__

![image](https://github.com/giaplee/rag-gemini/assets/4475732/91f034d7-3155-4ded-845c-946818d69f79)

---------------------------
## Dataset
* Name: vn_landmark.csv
* Desc: Dataset này chỉ là một dataset nhỏ để phục vụ cho bài thực hành này, nó bảo gồm hơn 100 các địa danh nổi tiếng của Việt Nam.
* Columns: Name, Description, Province
* Bạn có thẻ cập nhật (làm giàu thêm) dữ liệu cho 1 địa danh bằng cách thêm dữ liệu vào Description

## Encoder (vectorize)
* Ở đây chúng ta sử dụng **sentence-transformers** và pre-trained model: **all-MiniML-L6-v2** từ __HuggingFace__
* Bạn có thể tham khảo thêm thông tin tại đây <a href="https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2" target="_blank">https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2</a>
* Ở đây tôi lựa chọn **all-MiniML-L6-v2** vì nó là một model nhỏ gọn chỉ 80MB nhưng có performance rất tốt cho sentence embedding
  cũng như Semantic Search. Bạn hãy xem chi tiết về các model phổ biến và performance của nó ở bảng phía dưới (sau đó bạn có thể thử nghiệm
  với các model hình khác cho performance tốt hơn).
![image](https://github.com/giaplee/rag-gemini/assets/4475732/8d13247a-01ff-44fb-b1c3-adb58e8530c4)

## Run Qdrant in Docker
**Dứoi đây là phần hướng dẫn cài đặt và khởi chạy cũng như quản lí dataset/collection của bạn với Qdrant**
(Tôi tạm để English để bạn đọc cho chính xác >> và sẽ translate sang Vietnamese sau khi có bài hướng dẫn chi tiết hơn :) )
You need to manage all of your data using a vector engine. Qdrant lets you store, update or delete created vectors. Most importantly, it lets you search for the nearest vectors via a convenient API.

Dưới đây là 2 bước cơ bản để cài đặt và khởi chạy Qdrant

__Download the Qdrant image from DockerHub.__
``docker pull qdrant/qdrant``

__Start Qdrant inside of Docker.__
``docker run -p 6333:6333 \
    -v $(pwd)/qdrant_storage:/qdrant/storage \
    qdrant/qdrant``

__You should see output like this__

`...
[2021-02-05T00:08:51Z INFO  actix_server::builder] Starting 12 workers
[2021-02-05T00:08:51Z INFO  actix_server::builder] Starting "actix-web-service-0.0.0.0:6333" service on 0.0.0.0:6333`

Sau đó bạn truy cập vào địa chỉ localhost trên máy tính của bạn: http://localhost:6333/. Bạn sẽ được đưa tới giao diện của Qdrant.
![image](https://github.com/giaplee/rag-gemini/assets/4475732/4690adc4-c6aa-45d3-ac59-209e8b441d71)
Và bạn có thể xem chi tiết một collection như dao diện phía dưới
![image](https://github.com/giaplee/rag-gemini/assets/4475732/ffec60bf-2966-4782-9465-3cce4f547dfb)

Tất cả data được upload tới Qdrant được lưu trữ bên trong path này `./qdrant_storage` (thư mục được vẫn được giữ lại ngay cả khi bạn tạo lại container)

## Hãy tạo API key của bạn với Google AI Studio:
* Truy cập vào đường link này https://aistudio.google.com/app/prompts/new_chat hoặc https://ai.google.dev/aistudio
* Sau đó hãy login bằng acc gmail của bạn
 ![image](https://github.com/giaplee/rag-gemini/assets/4475732/9fc6f761-e0a3-48c4-a0e5-2d845dfe6b28)
* Sau đó chọn `Get API key` >> và chọn tiếp `Create API Key` nếu bạn chưa tạo 1 key nào trước đây hoặc đã xoá nó
* Và bạn có thể thử nhanh bằng cUrl command và call trực tiếp tới API endpoint như hình dưới (theo tôi biết thì hiện tại việc sử dụng Gemini là FREE):
  ![image](https://github.com/giaplee/rag-gemini/assets/4475732/238a764a-80b4-4287-a26a-20927168f6d2)

## Gemini Cookbook
* Là nơi bạn có thể tìm hiểu thêm cách sử dụng Gemini từ cơ bản đến nâng cao với link này
  <a href="https://github.com/google-gemini/cookbook" target="_blank">https://github.com/google-gemini/cookbook</a>
  
