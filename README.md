

# 前言




  在OSG中，对于一些效果未被选中或者包含等业务，需要半透明效果来实现。  本篇描述OSG的半透明实现方式。



 

# Demo




  ![请添加图片描述](https://img2024.cnblogs.com/blog/1971530/202412/1971530-20241212144111469-54622335.gif)



 

# 透明




## 功能概述




  透明效果在三维场景中扮演着重要角色，它能够模拟玻璃、水体、烟雾等自然现象，增加场景的层次感和真实感。然而，透明效果的实现并非易事，它涉及到复杂的渲染技术和算法。OSG作为一个功能强大的场景图库，为透明效果的实现提供了强有力的支持。




## 材质属性的调整




  在OSG中，实现透明效果的第一步是调整材质属性。材质属性决定了物体表面的外观特性，包括颜色、光泽度、反射率和透明度等。要实现透明效果，需要设置材质的透明度属性。  OSG中的osg::Material类用于设置物体的材质属性。通过调整osg::Material::TRANSPARENCY属性，我们可以控制物体的透明度。同时，我们还需要设置物体的颜色属性，并指定颜色的RGBA分量，其中A分量表示透明度。




## 深度测试的设置




  深度测试是三维渲染中的一项重要技术，它用于确定物体在场景中的前后关系。在实现透明效果时，深度测试的设置尤为关键。需要确保深度测试是开启的，以便正确处理透明物体与背景或其他物体的遮挡关系。然而，由于透明物体具有部分遮挡的特性，还需要考虑深度写入（GL\_DEPTH\_WRITEMASK）的设置。在某些情况下，关闭深度写入可以避免透明物体渲染时的深度冲突问题。




## 渲染顺序的控制




  透明物体的渲染顺序对其最终呈现效果具有重要影响。为了获得正确的渲染效果，我们需要确保透明物体按照从远到近的顺序进行渲染。OSG提供了透明排序机制来帮助我们实现这一目标。  通过设置osg::StateSet::TRANSPARENT\_BIN渲染提示，我们可以将透明物体添加到单独的渲染队列中。OSG将按照从远到近的顺序渲染这些物体，从而确保渲染结果的正确性。




## 混合模式的应用




  混合模式是实现透明效果的关键技术之一。它决定了透明物体与背景或其他物体混合时的颜色计算方式。在OSG中，我们可以通过设置osg::BlendFunc属性来指定混合模式。  常见的混合模式包括源颜色与目的颜色的加权和、源颜色与目的颜色的差值等。通过选择合适的混合模式，我们可以获得不同的透明效果。例如，使用GL\_SRC\_ALPHA和GL\_ONE\_MINUS\_SRC\_ALPHA作为混合因子，可以实现标准的透明度混合效果。  在OpenSceneGraph（OSG）中，实现透明效果通常涉及调整材质属性、深度测试设置以及渲染顺序。  要设置对象透明，是通过调整材质的透明度属性。osg::Material 类用于设置对象的材质属性，其中 osg::Material::TRANSPARENCY属性可以用于设置透明度。




## 基本实现流程




* 创建材质实例，通过材质实现的（不是常规思维RGBA，因为A在此无效）
* 材质实例设置材质颜色，材质颜色只有RGB有效，A无效
* 设置材质实例的透明度
* 获取模型（需要透明）的模型状态集
* 状态集开启模型的深度测试
* 状态集设置透明通道单独渲染
* 状态集设置混合设置模式




## 注意事项




* 确保透明对象在渲染队列中的顺序是正确的。OSG的透明排序机制可以帮助处理这个问题，但在某些复杂场景中，你可能需要手动控制渲染顺序。
* 深度写入（GL\_DEPTH\_WRITEMASK）和深度测试（GL\_DEPTH\_TEST）的设置会影响透明对象的渲染效果。
* 混合模式（osg::BlendFunc）的设置会影响透明对象与背景或其他对象的混合方式。




  通过上述步骤，应该能够在OpenSceneGraph中实现基本的透明效果。如果需要更高级的透明处理，可以进一步探索OSG的渲染队列和混合模式设置。



 

# 透明实现步骤




## 步骤一：获取状态集




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202412/1971530-20241212144110636-1748573259.png)





```
// 步骤一：获取状态集
osg::ref_ptr pStateSet = pNode->getOrCreateStateSet();

```



## 步骤二：开启深度测试




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202412/1971530-20241212144110692-1884106373.png)





```
// 步骤二：状态集 设置深度测试开启，确保透明的物体深度测试开启
pStateSet->setMode(GL_DEPTH_TEST, osg::StateAttribute::ON);

```



## 步骤三：创建材质实例




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202412/1971530-20241212144110602-1869383355.png)





```
// 步骤三：创建材质实例
osg::ref_ptr pMaterial = new osg::Material;

```



## 步骤四：设置材质颜色（理论上这的a无效）




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202412/1971530-20241212144110563-1364640753.png)





```
// 步骤四：材质实例 设置材质颜色（RGB部分），透明度在颜色数组中设置
pMaterial->setDiffuse(osg::Material::FRONT_AND_BACK, osg::Vec4(color.x, color.y, color.z, color.a));

```



## 步骤五：设置材质透明度（理论上由这里控制透明度）




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202412/1971530-20241212144110663-654743680.png)





```
// 步骤五：材质实例 设置透明度（0-255）: 设置了反倒没图形了
pMaterial->setTransparency(osg::Material::FRONT_AND_BACK, color.a * 255.0);
//    pMaterial->setTransparency(osg::Material::FRONT_AND_BACK, 255.0);

```



## :[FlowerCloud机场](https://hanlianfangzhi.com)步骤六：设置材质




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202412/1971530-20241212144110629-532785711.png)





```
// 步骤六：状态集 设置材质
pStateSet->setAttributeAndModes(pMaterial.get());

```



## 步骤七：设置透明通道单独渲染




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202412/1971530-20241212144110570-1654524797.png)





```
// 步骤七：状态集 设置透明通道单独渲染
pStateSet->setRenderingHint(osg::StateSet::TRANSPARENT_BIN);

```



## 步骤八：设置渲染混合模式




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202412/1971530-20241212144110605-233165747.png)





```
// 步骤八：状态集 设置渲染混合模式
pStateSet->setAttributeAndModes(new osg::BlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA));

```


 

# Demo源码




## OsgManager.cpp相关函数代码





```
osg::ref_ptr OsgManager::createSphere(Point3F center, double radius, double ratio)
{
    // 绘制球体
    // 步骤一：创建一个用户保存几何信息的对象osg::Geode
    osg::ref_ptr pGeode = new osg::Geode;
    // 步骤二：创建专门指明精细度的类osg::TessellationHints，并设置对应精细度
    osg::ref_ptr pHints = new osg::TessellationHints;
    pHints->setDetailRatio(ratio);
    // 步骤三：绘制几何类型(几何体)
    pGeode->addDrawable(new osg::ShapeDrawable(new osg::Sphere(osg::Vec3(center.x, center.y, center.y), radius), pHints));

    return pGeode.get();
}

osg::ref_ptr OsgManager::setTransparency(osg::Node *pNode, Point4F color)
{
#if 1
    // 设置透明度

    // 步骤一：获取状态集
    osg::ref_ptr pStateSet = pNode->getOrCreateStateSet();
    // 步骤二：状态集 设置深度测试开启，确保透明的物体深度测试开启
    pStateSet->setMode(GL_DEPTH_TEST, osg::StateAttribute::ON);
    // 步骤三：创建材质实例
    osg::ref_ptr pMaterial = new osg::Material;
    // 步骤四：材质实例 设置材质颜色（RGB部分），透明度在颜色数组中设置
    pMaterial->setDiffuse(osg::Material::FRONT_AND_BACK, osg::Vec4(color.x, color.y, color.z, color.a));
    // 步骤五：材质实例 设置透明度（0-255）: 设置了反倒没图形了
//    pMaterial->setTransparency(osg::Material::FRONT_AND_BACK, color.a * 255.0);
//    pMaterial->setTransparency(osg::Material::FRONT_AND_BACK, 255.0);
    // 步骤六：状态集 设置材质
    pStateSet->setAttributeAndModes(pMaterial.get());
    // 步骤七：状态集 设置透明通道单独渲染
    pStateSet->setRenderingHint(osg::StateSet::TRANSPARENT_BIN);
    // 步骤八：状态集 设置渲染混合模式
    pStateSet->setAttributeAndModes(new osg::BlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA));
//    static int z = 0;
//    pStateSet->setRenderBinDetails(z++,QString("RenderBin%1").arg(z).toStdString());
#else
    osg::ref_ptr pMaterial = new osg::Material;
    // Alpha混合开启
    osg::ref_ptr pStateSet = pNode->getOrCreateStateSet();
    //取消深度测试
    pStateSet->setMode(GL_BLEND,osg::StateAttribute::ON);
    pStateSet->setMode( GL_DEPTH_TEST, osg::StateAttribute::OFF  );
    pStateSet->setMode( GL_LIGHTING, osg::StateAttribute::OFF | osg::StateAttribute::PROTECTED );
    pStateSet->setRenderBinDetails(11, "RenderBin");
#endif
    return pMaterial.get();
}

```



## OsgWidget.cpp





```
osg::ref_ptr OsgWidget::getTransparency()
{
    // 其他demo的控件
    updateControlVisible(false);

    osg::ref_ptr pGroup = new osg::Group();
    {
        // 创建几何体
        osg::ref_ptr pGeode = OsgManager::createSphere(Point3F(0, 0, 0), 0.5);
        // 设置透明度
        osg::ref_ptr pMaterial = OsgManager::setTransparency(pGeode, Point4F(1.0, 1.0, 1.0, 0.8));

        pGroup->addChild(pGeode);
    }
#if 0
    {
        // 创建几何体
        osg::ref_ptr pGeode = OsgManager::createSphere(Point3F(-1, 0, 0), 0.5);
        // 设置透明度
        osg::ref_ptr pMaterial = OsgManager::setTransparency(pGeode, Point4F(1.0, 0.0, 0.0, 0.25));

        pGroup->addChild(pGeode);
    }
    {
        // 创建几何体
        osg::ref_ptr pGeode = OsgManager::createSphere(Point3F(1, 0, 0), 0.5);
        // 设置透明度
        osg::ref_ptr pMaterial = OsgManager::setTransparency(pGeode, Point4F(0.0, 1.0, 0.0, 0.25));

        pGroup->addChild(pGeode);
    }
    {
        // 创建几何体
        osg::ref_ptr pGeode = OsgManager::createSphere(Point3F(0, -1, 0), 0.5);
        // 设置透明度
        osg::ref_ptr pMaterial = OsgManager::setTransparency(pGeode, Point4F(0.0, 0.0, 1.0, 0.50));


        pGroup->addChild(pGeode);
    }
    {
        // 创建几何体
        osg::ref_ptr pGeode = OsgManager::createSphere(Point3F(0, 1, 0), 0.5);
        // 设置透明度
        osg::ref_ptr pMaterial = OsgManager::setTransparency(pGeode, Point4F(1.0, 1.0, 0.0, 0.50));
        pGroup->addChild(pGeode);
    }
#endif

    {
        // 创建几何体
        osg::ref_ptr pGeode = OsgManager::createSphere(Point3F(0, 0, -1), 0.5);
        // 设置透明度
        osg::ref_ptr pMaterial = OsgManager::setTransparency(pGeode, Point4F(1.0, 0.0, 1.0, 0.5));
        pGroup->addChild(pGeode);
    }
    {
        // 创建几何体
        osg::ref_ptr pGeode = OsgManager::createSphere(Point3F(0, 0, 1), 0.5);
        // 设置透明度
        osg::ref_ptr pMaterial = OsgManager::setTransparency(pGeode, Point4F(0.0, 1.0, 1.0, 0.5));
        pGroup->addChild(pGeode);
    }

    // 开启深度测试
//    OsgManager::setDepthTest(pGroup, true);

    // 关闭光照
//    OsgManager::setLighting(pGroup.get(), false);

    return pGroup.get();
}

```


 

# 工程模板v1\.39\.0




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202412/1971530-20241212144110571-575515712.png)



 

# 入坑




## 入坑一：设置透明后不显示




### 问题




  设置透明后不显示  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202412/1971530-20241212144110626-893890884.png)




### 尝试




  去掉透明度设置后，可以显示：  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202412/1971530-20241212144110702-1023132259.png)




  设置后就不显示，检查代码设置流程，并没有发现问题，然后查看了Demo代码，半透明也不显示；  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202412/1971530-20241212144110615-2119066436.png)




  到现在为止，笔者osg3\.4\.0的ming32版本种，旋转中心和半透明都有问题。  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202412/1971530-20241212144110531-1501490488.png)




  然后继续测试，发现设置透明度没用，但是设置透明颜色可以：  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202412/1971530-20241212144110731-125126029.png)




### 解决




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202412/1971530-20241212144110583-1994615716.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202412/1971530-20241212144110620-1577185774.png)




## 入坑二：出现渲染截面




### 问题




  出现渲染截面，测试只有球体、球面的时候才出现。  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202412/1971530-20241212144110627-19873323.png)




  换个颜色：  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202412/1971530-20241212144110653-415895989.png)




### 原理




  这是深度测试问题，单独开了每一个的深度测试，需要开这几个模型进行深度测试，开了深度测试也是一样，检查总代码是开了的，尝试下关闭所有深度测试，启动就有问题（开启深度测试，至少启动没有问题）：  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202412/1971530-20241212144110670-2134319490.png)




  开启深度测试，关闭光照：  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202412/1971530-20241212144110624-1663383203.png)




  再次尝试打开stl球体模型，也是不行的，效果跟上面的一样，下面是绘制的stl球体：  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202412/1971530-20241212144110703-1839406996.png)




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202412/1971530-20241212144110715-239587458.png)




### 解决




  未解决，准备更换版本测试，经过多个版本都是一样。



