```py
# 加载数据集
import numpy as np
import os
import cv2


def load_data_by_opencv():
    flat_data = []

    # 遍历目录下的所有图片
    file_dir = 'E:\\1_practices\\'
    for root, dirs, files in os.walk(file_dir):
        for file in files:
            if os.path.splitext(file)[1] == '.png':
                path = file_dir + file
                # COLOR_RGB2GRAY：指定用灰度图像的方式打开图片，即将原始图像转化为灰度图像再打开
                src = cv2.imread(path, cv2.IMREAD_GRAYSCALE)
                # 将src转成类型float的数组
                src = np.array(src, dtype='float')
                # 转成一维数组
                src = src.flatten()
                flat_data.append(src)

        flat_data = np.array(flat_data, dtype='float')
        return flat_data


def load_targets():
    flat_data = load_data_by_opencv()
    target = [1, 0, 1, 0, 1, 0, 1, 1, 1, 0]
    return {'data': flat_data, 'target': target}
```