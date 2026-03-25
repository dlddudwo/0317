# ONNX Runtime 모델 로드 확인 예시

아래 코드는 `Ort::Session` 생성이 정상인지 확인하고,
추가로 입력/출력 메타데이터를 조회해 실제 모델이 로드되었는지 검증하는 예시입니다.

```cpp
#include <onnxruntime_cxx_api.h>
#include <iostream>
#include <string>

bool TryCreateSession(
    Ort::Env& env,
    const std::wstring& model_path,
    std::unique_ptr<Ort::Session>& out_session)
{
    try
    {
        Ort::SessionOptions opts;
        opts.SetGraphOptimizationLevel(GraphOptimizationLevel::ORT_ENABLE_EXTENDED);

        out_session = std::make_unique<Ort::Session>(env, model_path.c_str(), opts);

        // 기본 로드 검증: 입/출력 개수 확인
        size_t input_count = out_session->GetInputCount();
        size_t output_count = out_session->GetOutputCount();

        if (input_count == 0 || output_count == 0)
        {
            std::cerr << "[ERR] Session created but model IO is empty."
                      << " input=" << input_count
                      << " output=" << output_count << std::endl;
            out_session.reset();
            return false;
        }

        std::cout << "[OK] Model loaded."
                  << " input=" << input_count
                  << " output=" << output_count << std::endl;
        return true;
    }
    catch (const Ort::Exception& ex)
    {
        std::cerr << "[ERR] ONNX Runtime Exception: "
                  << ex.what()
                  << " | code=" << ex.GetOrtErrorCode() << std::endl;
        out_session.reset();
        return false;
    }
    catch (const std::exception& ex)
    {
        std::cerr << "[ERR] std::exception: " << ex.what() << std::endl;
        out_session.reset();
        return false;
    }
}

int main()
{
    Ort::Env env(ORT_LOGGING_LEVEL_WARNING, "model-check");

    std::wstring wstr_Image = L"model.onnx";
    std::unique_ptr<Ort::Session> session_image;

    if (!TryCreateSession(env, wstr_Image, session_image))
    {
        return 1;
    }

    // 여기까지 오면 로드 성공
    // session_image->Run(...) 으로 추가 런타임 검증 가능
    return 0;
}
```

## 질문에 주신 형태를 그대로 쓴 최소 체크

```cpp
Ort::Session* session_image_ptr = nullptr;
try {
    session_image_ptr = new Ort::Session(env_Image, wstr_Image.c_str(), Ort::SessionOptions{});
    std::cout << "[OK] model load success\n";
    std::cout << "inputs=" << session_image_ptr->GetInputCount()
              << ", outputs=" << session_image_ptr->GetOutputCount() << "\n";
} catch (const Ort::Exception& e) {
    std::cerr << "[ERR] model load fail: " << e.what() << "\n";
}
```

> 메모리 안전성 때문에 가능하면 raw pointer(`new`)보다 `std::unique_ptr<Ort::Session>` 사용을 권장합니다.
