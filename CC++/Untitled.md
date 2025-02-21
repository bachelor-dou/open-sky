onnxruntime中的子图（sub_graph）定义分为两类：

```c++
onnxruntime/core/framework/graph_partitioner.cc      GraphPartitioner::Partition
1. “子图”是当前图形实例中节点的子集。
2. 控制流节点有嵌套的图形实例，也称为子图、 但它们是完全独立的图实例，而不是单个图实例中的节点子集。
```

​	onnxruntime图优化其实就是将构建的图进行转换，主要包括子图简化、节点消除、更复杂的节点融合和布局优化等；

图优化分为三个等级：

```
enum class TransformerLevel : int {
  Default = 0,  // required transformers only
  Level1,       // basic optimizations
  Level2,       // extended optimizations
  Level3,       // layout optimizations
  // The max level should always be same as the last level.
  MaxLevel = Level3
};


# session_options.h 
// set graph optimization level  默认3个等级全部开启
  TransformerLevel graph_optimization_level = TransformerLevel::Level3;
```

graphtransformer调用流程如下：

```c++
InferenceSession::Initialize() // 在session  init过程中，执行graph-transformers
	InferenceSession::AddPredefinedTransformers
		GenerateTransformers
			case TransformerLevel::Level1:
				GenerateRuleBasedGraphTransformer
					GenerateRewriteRules
						case TransformerLevel::Level1:
							EliminateIdentity
							CastElimination
							ConvAddFusion
							......
				ConstantSharing
				CommonSubexpressionElimination
				ConstantFolding
				....
			case TransformerLevel::Level2:
				GemmActivationFusion
				MatMulIntegerToFloatFusion
				ConvActivationFusion
				....
			case TransformerLevel::Level3:
				NchwcTransformer
				NhwcTransformer
```









```
#include <iostream>
#include <vector>

#include "onnxruntime_cxx_api.h"

// path of model, Change to user's own model path
const char* model_path = "./onnx/resnet50_Opset16.onnx";

/**
 * @brief Input data preparation provided by user.
 *
 * @param num_input_nodes The number of model input nodes.
 * @return  A collection of input data.
 */
std::vector<std::vector<float>> input_prepare(size_t num_input_nodes) {
  std::vector<std::vector<float>> input_datas;
  input_datas.reserve(num_input_nodes);

  constexpr size_t input_data_size = 3 * 224 * 224;
  std::vector<float> input_data(input_data_size);
  // initialize input data with values in [0.0, 1.0]
  for (unsigned int i = 0; i < input_data_size; i++)
    input_data[i] = (float)i / (input_data_size + 1);
  input_datas.push_back(input_data);

  return input_datas;
}

/**
 * @brief Model output data processing logic(For User updates).
 *
 * @param output_tensors The results of the model output.
 */
void output_postprocess(std::vector<Ort::Value>& output_tensors) {
  auto floatarr = output_tensors.front().GetTensorMutableData<float>();

  for (int i = 0; i < 5; i++) {
    std::cout << "Score for class [" << i << "] =  " << floatarr[i] << '\n';
  }
  
  std::cout << "Done!" << std::endl;
}

/**
 * @brief The main functions for model inference.
 *
 *  The complete model inference process, which generally does not need to be
 * changed here
 */
void inference() {
  const auto& api = Ort::GetApi();

  // Enable cann graph in cann provider option.
  OrtCANNProviderOptions* cann_options = nullptr;
  api.CreateCANNProviderOptions(&cann_options);

  // Configurations of EP
  std::vector<const char*> keys{
      "device_id",
      "npu_mem_limit",
      "arena_extend_strategy",
      "enable_cann_graph"};
  std::vector<const char*> values{"0", "4294967296", "kNextPowerOfTwo", "1"};
  api.UpdateCANNProviderOptions(
      cann_options, keys.data(), values.data(), keys.size());

  // Convert to general session options
  Ort::SessionOptions session_options;
  api.SessionOptionsAppendExecutionProvider_CANN(
      static_cast<OrtSessionOptions*>(session_options), cann_options);

  Ort::Session session(Ort::Env(), model_path, session_options);

  Ort::AllocatorWithDefaultOptions allocator;

  // Input Process
  const size_t num_input_nodes = session.GetInputCount();
  std::vector<const char*> input_node_names;
  std::vector<Ort::AllocatedStringPtr> input_names_ptr;
  input_node_names.reserve(num_input_nodes);
  input_names_ptr.reserve(num_input_nodes);
  std::vector<std::vector<int64_t>> input_node_shapes;
  std::cout << num_input_nodes << std::endl;
  for (size_t i = 0; i < num_input_nodes; i++) {
    auto input_name = session.GetInputNameAllocated(i, allocator);
    input_node_names.push_back(input_name.get());
    input_names_ptr.push_back(std::move(input_name));
    auto type_info = session.GetInputTypeInfo(i);
    auto tensor_info = type_info.GetTensorTypeAndShapeInfo();
    input_node_shapes.push_back(tensor_info.GetShape());
  }

  // Output Process
  const size_t num_output_nodes = session.GetOutputCount();
  std::vector<const char*> output_node_names;
  std::vector<Ort::AllocatedStringPtr> output_names_ptr;
  output_names_ptr.reserve(num_input_nodes);
  output_node_names.reserve(num_output_nodes);
  for (size_t i = 0; i < num_output_nodes; i++) {
    auto output_name = session.GetOutputNameAllocated(i, allocator);
    output_node_names.push_back(output_name.get());
    output_names_ptr.push_back(std::move(output_name));
  }

  //  User need to generate input date according to real situation.
  std::vector<std::vector<float>> input_datas = input_prepare(num_input_nodes);

  auto memory_info = Ort::MemoryInfo::CreateCpu(
      OrtAllocatorType::OrtArenaAllocator, OrtMemTypeDefault);

  std::vector<Ort::Value> input_tensors;
  input_tensors.reserve(num_input_nodes);
  for (size_t i = 0; i < input_node_shapes.size(); i++) {
    auto input_tensor = Ort::Value::CreateTensor<float>(
        memory_info,
        input_datas[i].data(),
        input_datas[i].size(),
        input_node_shapes[i].data(),
        input_node_shapes[i].size());
    input_tensors.push_back(std::move(input_tensor));
  }

  auto output_tensors = session.Run(
      Ort::RunOptions{nullptr},
      input_node_names.data(),
      input_tensors.data(),
      num_input_nodes,
      output_node_names.data(),
      output_node_names.size());

  // Processing of out_tensor
  output_postprocess(output_tensors);
}

int main(int argc, char* argv[]) {
  inference();
  return 0;
}
```

