### Vulkan Tutorial

#### 同步问题

`drawFram()`中

1. CPU调用`vkAcquireNextImageKHR`，得知往哪个`framebuffer`中画，需要等待`imageAvailableSemaphores`；

2. CPU得到`vkAcquireNextImageKHR`返回的`index`之后，准备向GPU提交渲染命令：调用`vkQueueSubmit`，把`index`对应的`command buffer`提交到`queue`里，然后GPU就会读`queue`里的内容然后执行;

3. 执行结束之后，发出`renderFinished`信号，调用之前记录的`vkQueuePresentKHR`，在参数里指定读取的`index`，就可以让presentation engine读取image然后显示了;

不同MAX_FRAMES_IN_FLIGHT下的执行情况:

![](.\assets\Snipaste_2024-09-06_11-31-55.png)

CPU处理一帧，GPU渲染一帧，然后CPU再处理下一帧，CPU与GPU之间会相互等待，这样做确实会造成性能的损失。如果我们预渲染的帧数足够多，GPU就能一直被源源不断的任务填满。但是这样的问题在于用户是看到了屏幕显示的内容之后再作出的反应，如果预渲染太多帧，用户作出反应的后面很多帧还是之前提前提交的没有考虑用户输入的预渲染帧（FIFO模式的`swap chain`保证先提交的`present`命令会先被`present`），于是输入的延迟就会变得很大。

我们选择预渲染一帧，也就是`MAX_FRAMES_IN_FLIGHT = 2`，注意这些帧是虚拟的，我们需要通过`vkAcquireNextImageKHR`对应上swap chain中的image。我们的基本逻辑是把预渲染帧当成一个ring buffer，用`current_frame`追踪渲染任务提交到了哪一帧，然后在循环的最后更新`current_frame=(current_frame+1)%MAX_FRAMES_IN_FLIGHT`。上面简单情况里的各种同步机制都需要为每一个预渲染帧准备一份，包括`image_available`、`render_finished`两个semaphore，还有标记渲染完成的fence：`in_flight_fence`。这个fence需要传入`vkQueueSubmit`中，表示渲染完成之后打开这个fence，然后在循环的最开始用`vkWaitForFences`等待这个fence，这样做等同于上面的`vkQueueWaitIdle`的作用，只不过现在是对每一个预渲染帧都有一个单独的fence。在程序最开始的时候我们会直接遇到`vkWaitForFences`，因此fence在创建的时候必须置为打开状态。与semaphore不同，`vkWaitForFences`之后fence的状态不会改变，于是还需要通过`vkResetFences`重置fence。

到这里实际上就完成了预渲染的实现，但是在Tutorial里还考虑了更细致的情况。如果`MAX_FRAMES_IN_FLIGHT`里的image数量比swap chain里的更多，那我们还是会遇到跟之前一样的情况（假想`MAX_FRAMES_IN_FLIGHT`很大很大，CPU的行为还是不管GPU不断提交任务）。为了避免这种情况出现，我们还需要为swap chain里每个image再增加一个记录使用情况的fence：`image_in_flight`。Tutorial里的实现比较神奇，`image_in_flight`实际上并不是创建出来的，而是从`in_flight_fence`里复制来的。当预渲染帧（虚拟）通过`vkAcquireNextImageKHR`得到了一个image，就把对应的`in_flight_fence`复制到对应的`image_in_flight`。在后面的循环中如果`vkAcquireNextImageKHR`给出了跟之前一样的`image index`（通过检查`image_in_flight`不为空），就先等之前的任务完成完，也就是等待这个`image_in_flight`的fence。好，现在的情况是同一个fence可能在最开始的`in_flight_fence`的地方被block住，也可能在`image_in_flight`的地方被block住，如果我们在第一个`in_flight_fence`的地方等到了然后立刻`vkResetFences`，运行到`image_in_flight`的时候还是会被block住，此时完全没有机会被解锁，所以最好的做法是把`vkResetFences`从第一次等待之后移动到`vkQueueSubmit`之前，这样就不会出现任何问题了。

#### Descriptor

Descriptor是一个不透明的数据结构，代表一个着色器资源，如Buffer、Buffer View、Imgae、Sampler、Combined Image Sampler。

Descriptor会被组织成DescriptorSet，在Command Record过程中被绑定，以便在随后的DrawCall中使用。每个DescriptorSet的内容安排是由DescriptorSet Layout决定的，它决定了哪些Descriptor可以被存储在其中。

Binding Model：

- DescriptorSetLayout由Descriptor Binding组成，会控制每个Descriptor的排列方式。
- Descriptor Binding将Descriptor和shader进行绑定，给shader提供了对资源操作的接口。
- DescriptorSet需要指定DescriptorSetLayout最后由DescriptorPool来真正分配出来。
- 通过VkWriteDescriptorSet/VkCopyDescriptorSet来真正将资源绑定或者更新数据。
- PipelineLayout通过DescriptorSetLayout以及PushConst创建并完成整个PipeLine的数据绑定。
- 最后使用vkCmdBindDescriptorSets完成对于DescriptorSet的绑定。

#### RenderPass & Attachment &Framebuffer

- Attachment只是资源描述(元数据)定义一些加载/存储操作以及相应资源的格式

- 一个Attachment对应于一个VkImageView

- RenderPass可以通过Framebuffer来获得实际的Image

- RenderPass其实是通过Framebuffer中包含的ImageView拿到真正的数据

RenderPass需要与Framebuffer获取到真正的Image才可以渲染出真正的结果。Framebuffer代表了RenderPass所使用的特定内存的集合，也就是Attachment真正对应的内存。已经了解到RenderPass只是元数据，真正的Image需要从Framebuffer中获取。Framebuffer会定义了哪个ImageView是对应到RenderPass的哪个Attachment。ImageView定义了要使用Image的哪一部分。Image定义了哪个物理内存被使用以及Texel的格式。

### qReplay

`rp_execf_vk.cpp`

```cpp
// 调整缓冲区中的加速结构地址
RP_ERR API_0_axvAdjustBufAccAddressInCmd(RP_Exec* pEc)
{
    //todo: need to delete
    //RP_PRINT_LOG(pEc->m_pCore, "Info: Line %d, axvAdjustBufAccAddressInCmd\n", pEc->m_pCore->m_pLang->GetLineIdx());

    auto rpr = RP_ERR_SUCCESS;

    // 获取参数
    const auto cmdBuf = (VkCommandBuffer)pEc->GetArgHwV(0);
    RP_ID cbId = pEc->GetArg(0);
    const auto pCB = pEc->m_pCore->m_pLang->m_CmdBufs_VK.GetCB(cbId);
    VkBuffer buffer = (VkBuffer)pEc->GetArgHwV(1);
    VkDeviceSize offset = pEc->GetArg(2);
    VkDeviceSize size = pEc->GetArg(3);
    const auto fileName = (char*)(void*)pEc->GetArg(4);

    // 读取文件数据
    void* data = nullptr;
    size_t fileSize = 0;
    rpr = pEc->m_pCore->m_pLang->Read_bin(fileName, 0, 0, 0, &data, 0, 0, &fileSize);
    if (rpr != RP_ERR_SUCCESS)
    {
        RP_Free(data);
        return pEc->m_pCore->SetErr(RP_ERR_FAIL, nullptr, nullptr);//err code must be RP_ERR_FAIL, but keep errMsg & errLoc from Read_bin;
    }

    // 查找缓冲区和设备
    const auto iterBuf = pEc->m_pCore->m_pLang->m_Bufs_VK.find(pEc->GetArg(1));
    RP_ASSERT(iterBuf != pEc->m_pCore->m_pLang->m_Bufs_VK.end());
    const auto device = (VkDevice)pEc->GetHwVal(iterBuf->second->m_DevId);
    RP_Dev_VK* pDevVK = pEc->GetDevice_VK(device);

    // 创建临时缓冲区
    VkBuffer newBuf = VK_NULL_HANDLE;
    VkDeviceMemory newBufMem = VK_NULL_HANDLE;

    pEc->CreateTmpBufForCopy_VK(pDevVK, size, RP_TRUE, newBuf, newBufMem);

    // 映射内存并调整加速结构地址
    char* pData = nullptr;
    const VkResult vkr = pEc->m_VK.vkMapMemory(device, newBufMem, 0, size, 0, (void**)&pData);
    if (vkr == VK_SUCCESS)
    {
        memcpy(pData, data, (size_t)size);
        size_t primitiveCount = size / sizeof(VkAccelerationStructureInstanceKHR);
        //RP_PRINT_LOG(pEc->m_pCore, "Info: primitiveCount %d\n", primitiveCount);
        for (uint32_t i = 0; i < primitiveCount; i++)
        {
            VkAccelerationStructureInstanceKHR* pAccInstance = (VkAccelerationStructureInstanceKHR*)pData + i;
            uint64_t deviceAddress = pAccInstance->accelerationStructureReference;
            if (deviceAddress == 0)
            {
                RP_PRINT_LOG(pEc->m_pCore, "Info: pAccInstance->accelerationStructureReference is 0\n");
                continue;
            }
            const auto iterHandle = pEc->m_pCore->m_pLang->m_AddressToAccKHR_VK.find(deviceAddress);
            RP_ASSERT(iterHandle != pEc->m_pCore->m_pLang->m_AddressToAccKHR_VK.end());
            RP_ID accId = iterHandle->second;
            const auto& iter = pEc->m_AccKHRs_VK.find(accId);
            RP_ASSERT(iter != pEc->m_AccKHRs_VK.end());
            pAccInstance->accelerationStructureReference = iter->second.m_AccKHRAddress;
            //RP_PRINT_LOG(pEc->m_pCore, "Info: adjust pAccInstance->accelerationStructureReference from 0x%llx to 0x%llx\n", deviceAddress, pAccInstance->accelerationStructureReference);
        }
        pEc->m_VK.vkUnmapMemory(device, newBufMem);
    }
    else
    {
        RP_Free(data);
        return pEc->m_pCore->SetErr(RP_ERR_FAIL, "vkMapMemory fail !", __FUNCTION__);
    }

    VkBufferCopy region;
    region.srcOffset = 0;
    region.dstOffset = offset;
    region.size = size;

    // 拷贝数据
    const VkBufferMemoryBarrier lastDstBarrier = pEc->GetBufLastBarrier_VK(RP_INVALID_ID, pEc->GetArg(1));
    pEc->CopyBufInCB_VK(cmdBuf, newBuf, buffer, &region, nullptr, &lastDstBarrier);

    RP_Free(data);
    return rpr;
}
```
