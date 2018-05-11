copied data from google drive to gamone:
```
ssh gamone.whoi.edu
cd /usgs/data0/rsignell
rclone sync gdrive:/crs_maps .
```

made a small bash script to convert TIFs to cloud-optimized tifs (just doing the tiling and compresssion, skipping the overviews):
```
for file in *.tif
do
  gdal_translate $file ./cogeo/$file -co TILED=YES -co COMPRESS=LZW
done
```
Then copy these cloud-optimized geotiffs to S3 storage using Rclone:
```
rclone sync ./cogeo s3:rsignell/crs_maps
```
