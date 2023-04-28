layout: page
title: "使用微软edge web控件WebView2对外部图像数据进行高性能显示的办法"
permalink: /webview2_sharedbuffer_performance.md

# 使用微软edge web控件WebView2对外部图像数据进行高性能显示的办法

WebView2没有提供任何直接操控DOM的办法，因此想要让外部图像数据在网页中显示，有以下几种方式：

1. 内置一个标准web服务js通过fetch api进行请求完成任务。

2. 可以在宿主端对webResourceRequest进行uri请求拦截，而无需创建额外web服务，js也通过fetch api完成任务。

3. 宿主端把图片数据转换成base64编码的dataUrl，通过对页面注入js脚本完成，这个方法对于大图片来说性能会很差。

4. 使用webview提供的sharedbuffer机制（也就是宿主端和web渲染进程之间的共享内存），js得到arraybuffer后先转换成blob再得到dataUrl。

```
    getBlob(size: number, mime: string): Blob | null {
        if (SharedBuffer.sharedBuffer === null)
            return null;
        let buffer = new Uint8Array(SharedBuffer.sharedBuffer, 0, size);
        return new Blob([buffer], { type: mime });
    }

    getBlobUrl(size: number, mime: string): string | null {
        let blob = this.getBlob(size, mime);
        if (blob === null)
            return null;
        return URL.createObjectURL(blob);
    }
```

5. 性能最好的办法是创建canvas，然后宿主端把图片数据复制到sharedbuffer中，js端得到后把图像数据转换成ImageData，然后利用cavans的putImageData进行绘画，唯一要注意的是写入sharedbuffer数据必修是rgba32格式，也就是数据总字节数必须是4倍的长x宽。

```   
    getImageData(size: number, width: number, height: number): ImageData | null {
        if (SharedBuffer.sharedBuffer === null)
            return null;
        return new ImageData(new Uint8ClampedArray(SharedBuffer.sharedBuffer, 0, size), width, height);
    }
```
