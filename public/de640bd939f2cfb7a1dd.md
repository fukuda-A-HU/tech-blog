---
title: Web上からアップロードした画像のメタデータをサーバー側で取得する方法
tags:
  - Python
  - exif
  - Streamlit
private: false
updated_at: '2024-12-08T23:57:49+09:00'
id: de640bd939f2cfb7a1dd
organization_url_name: null
slide: false
ignorePublish: false
---

はじめまして、H-kunと申します。
A-kun ひとり Advent Calender 8日目です。
今回の話題は、Web上からアップロードした画像の撮影時刻・GPS情報を取得する方法について、解説します。

## やったこと

下記がExif情報を取得するモジュールです。exif_utils.pyとして保存します。
```python
import streamlit as st
from PIL import Image
from PIL.ExifTags import TAGS, GPSTAGS

def get_exif_data(image):
    """Extract Exif data from an image."""
    exif_data = {}
    info = image._getexif()
    if info:
        for tag, value in info.items():
            tag_name = TAGS.get(tag, tag)
            exif_data[tag_name] = value
    return exif_data

def get_geotagging(exif_data):
    """Extract GPS data from Exif."""
    if "GPSInfo" not in exif_data:
        return None
    
    gps_data = {}
    for tag, value in exif_data["GPSInfo"].items():
        tag_name = GPSTAGS.get(tag, tag)
        gps_data[tag_name] = value
    return gps_data

def get_decimal_coordinates(gps_data):
    """Convert GPS coordinates to decimal format."""
    def convert_to_decimal(coord, ref):
        degrees, minutes, seconds = coord
        decimal = degrees + (minutes / 60.0) + (seconds / 3600.0)
        if ref in ['S', 'W']:
            decimal = -decimal
        return decimal

    if gps_data:
        latitude = gps_data.get("GPSLatitude")
        latitude_ref = gps_data.get("GPSLatitudeRef")
        longitude = gps_data.get("GPSLongitude")
        longitude_ref = gps_data.get("GPSLongitudeRef")

        if latitude and latitude_ref and longitude and longitude_ref:
            return (
                convert_to_decimal(latitude, latitude_ref),
                convert_to_decimal(longitude, longitude_ref)
            )
    return None

def display_exif_info(uploaded_file):
    """Process and display Exif info from an uploaded image."""
    image = Image.open(uploaded_file)
    exif_data = get_exif_data(image)

    # 撮影時刻を取得
    datetime = exif_data.get("DateTimeOriginal", "情報なし")
    st.write(f"撮影時刻: {datetime}")

    # GPS情報を取得
    gps_data = get_geotagging(exif_data)
    if gps_data:
        coordinates = get_decimal_coordinates(gps_data)
        if coordinates:
            st.write(f"緯度: {coordinates[0]}")
            st.write(f"経度: {coordinates[1]}")
        else:
            st.write("GPS情報が見つかりませんでした。")
    else:
        st.write("GPS情報が見つかりませんでした。")
```

これをstreamlitで公開するコードが、下記になります。

```python
# FILE: view.py
import streamlit as st
from exif_utils import display_exif_info

# Streamlitアプリの構築
st.title("Exif情報取得アプリ")
st.write("画像をアップロードして、Exif情報を確認してください。")

uploaded_file = st.file_uploader("画像をアップロードしてください", type=["jpg", "jpeg", "png"])

if uploaded_file:
    display_exif_info(uploaded_file)
```

## Exifデータが取得できるか確認

確認したところ、条件によってはブラウザ側でExif情報が削除されてしまうことがわかりました。

動作検証端末

iPhone XR (iOS 17.6.1) 
- Chrome 121.0.6167.66 -> OK
- Safari -> OK

Android (Android 15)
- Chrome 131.0.6778.104 -> NG

## （代替策）Google Formを利用する

いくつか試してみたところ、Google Formを利用することで、Exif情報がついたままま画像のアップロードが可能であることがわかりました。
そこで、アップロードされた画像をGoogleDriveの特定のフォルダに保存し、定期的にそのフォルダの内容を監視することで、Exif情報を取得することができると考えられます。これはまた次回の課題としたいと思います。
