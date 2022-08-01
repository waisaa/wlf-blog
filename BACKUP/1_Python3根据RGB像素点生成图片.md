# [Python3根据RGB像素点生成图片](https://github.com/waisaa/wlf-blog/issues/1)

## Background
> - 通过合适的可视化手段，把监测数据以图的方式展现出来，能更直观的看出数据所反映的情况。例如下图，根据降雨量的划分颜色区间，然后把监测到的降雨量数据转换成RGB颜色值，最终通过不同颜色表示降雨量。
> - 这里我们用某个城市的降雨量监测数据来制作一个这样的图片。
> - 我们常见的RGB图像通常只有R、G、B三个通道，在图像处理的过程中会遇到往往需要向图像中添加透明度信息，如公司logo的设计，其输出图像文件就需要添加透明度，即需要在RGB三个通道的基础上添加alpha通道信息(0完全不透明，255完全透明)。

![在这里插入图片描述](https://img-blog.csdnimg.cn/c215c7b6e6664fa5ad97f93ae072539a.png)
## 填色生图
> - 这里的测试数据（积水深度数据）如下图所示，[原数据文件下载地址](https://pan.baidu.com/s/1cJfTBiledWgKj30HTNL54Q)，提取码：king
> - 我们给不同的监测数据设置了不同的颜色区间。

![在这里插入图片描述](https://img-blog.csdnimg.cn/8ef1f88d39054de1b85931f32db3c440.png)
- 最终生成的图片如下图所示

![在这里插入图片描述](https://img-blog.csdnimg.cn/f65c75ee63a743c5b225ef1a4e9d74dd.png)
- 源码
```python3
import pandas as pd
from PIL import Image, ImageDraw


def get_rgba_by_depth(depth: str):
    """根据积水深度获取RGBA值
    返回值：(r,g,b,a)
    """
    depth = float(depth)
    if depth == -9999.0:
        return (255, 255, 255, 0)
    elif 0 <= depth < 0.15:
        return (179, 204, 255, 255)
    elif 0.15 <= depth < 0.27:
        return (128, 153, 255, 255)
    elif 0.27 <= depth < 0.4:
        return (89, 128, 255, 255)
    elif 0.4 <= depth < 1.0:
        return (38, 115, 242, 255)
    elif depth >= 1.0:
        return (0, 77, 204, 255)


def draw_img(rgbas: list, width: int = 200, height: int = 150):
    """根据所给的RGBA值生成图片，默认宽200高150"""
    # 透明背景，RGBA值为：(0, 0, 0, 0)
    # img = Image.new('RGB', (width, height), (0, 0, 0))
    img = Image.new('RGBA', (width, height), (0, 0, 0, 0))
    # 填充像素
    img.putdata(rgbas)
    # 显示图片
    # img.show()
    # 保存图片
    img.save('pixel_img/hydrops.png')


def draw_img_by_point(rgbas: list, width: int = 200, height: int = 150):
    """根据所给的RGBA值生成图片，默认宽200高150"""
    # 透明背景，RGBA值为：(0, 0, 0, 0)
    # img = Image.new('RGB', (width, height), (0, 0, 0))
    img = Image.new('RGBA', (width, height), (0, 0, 0, 0))
    draw = ImageDraw.Draw(img)
    # 填充每个像素
    for x in range(width):
        for y in range(height):
            idx = x * width + y
            draw.point((x, y), fill=rgbas[idx])
    # 显示图片
    # img.show()
    # 保存图片
    img.save('pixel_img/hydrops.png')


def img_rgb(file):
    """读取文件，返回rgb集合、宽和高"""
    res = []
    # 先获取列数
    hdf = pd.read_csv(file, header=None, delim_whitespace=True, nrows=1)
    row, cols = hdf.shape
    # 对每个数值进行RGBA值转换
    converters = {i: get_rgba_by_depth for i in range(cols)}
    # 读取csv文件数据
    ds = pd.read_csv(file, header=None, delim_whitespace=True, converters=converters)
    # 指定读取前多少行
    # ds = pd.read_csv(dat, header=None, delim_whitespace=True, converters=converters, nrows=200)
    # 获取图片大小
    height, width = ds.shape
    # 组织图片生成需要的数据源
    for x in ds.index:
        for y in ds.columns:
            res.append(ds.loc[x, y])
    print(len(res), width, height)
    # 生成图片
    draw_img(res, width, height)
    # 第二种方法，当数据量较大时会很慢
    # draw_img_by_point(res, width, height)


def main():
    """主函数"""
    src_file = 'pixel_img/hydrops.dat'
    img_rgb(src_file)


if __name__ == '__main__':
    main()
```