package main

import (
	"flag"
	"fmt"
	"github.com/aliyun/aliyun-oss-go-sdk/oss"
	"log"
	"os"
	"time"
)

var (
	Help bool
	OssEndpoint string
	OssAccessKeyID string
	OssAccessKeySecret string
	OssBucketName string
	OssObjectPrefix string
	OssObjectName string
	OssPartSize int64
	LocalFile string
	Routines int
)

func init()  {
	flag.BoolVar(&Help, "help", false, "断点续传客户端使用帮助")
	flag.StringVar(&OssEndpoint, "ossEndpoint",
		"","输入OSS的Endpoint")
	flag.StringVar(&OssAccessKeyID, "accessKeyID", "","输入accessKeyID")
	flag.StringVar(&OssAccessKeySecret, "accessKeySecret", "","输入accessKeySecret")
	flag.StringVar(&OssBucketName, "bucketName", "","输入bucket name")
	flag.StringVar(&OssObjectPrefix, "objectPrefix", "","输入object前缀,注意以'/'结尾，比如TD/h5/")
	flag.StringVar(&OssObjectName, "objectName", "","输入object name,如果不输入，默认就是上传的文件名")
	flag.StringVar(&LocalFile, "localFile", "","输入object name")
	flag.Int64Var(&OssPartSize, "partSize", 100*1024*1024,"输入文件分片大小，默认值100M")
	flag.IntVar(&Routines, "routines", 3,"输入并发数，默认值为3")
	flag.Usage = usage
}

func usage() {
	_, _ = fmt.Fprintf(os.Stdout, `xyzqoss断点续传客户端: v1.0
Options:
`)
	flag.PrintDefaults()
}

func main() {

	flag.Parse()
	if Help {
		flag.Usage()
	} else {
		if OssObjectName == "" {
			OssObjectName = LocalFile
		}

		startTime := time.Now()

		log.Printf("将本地文件：%v 上传到oss的bucket：%v，对象名：%v",LocalFile,OssBucketName,OssObjectPrefix + OssObjectName)

		// 创建OSSClient实例。
		client, err := oss.New(OssEndpoint, OssAccessKeyID, OssAccessKeySecret)
		if err != nil {
			fmt.Println("创建OSSClient实例报错，Error:", err)
			os.Exit(-1)
		}

		// 获取存储空间。
		bucket, err := client.Bucket(OssBucketName)
		if err != nil {
			fmt.Println("获取bucket报错，Error:", err)
			os.Exit(-1)
		}

		// 设置分片大小为100 MB，指定分片上传并发数为3，并开启断点续传上传。
		// 其中<yourObjectName>与objectKey是同一概念，表示断点续传上传文件到OSS时需要指定包含文件后缀在内的完整路径，例如abc/efg/123.jpg。
		// "LocalFile"为filePath，100*1024为partSize。
		err = bucket.UploadFile(OssObjectPrefix + OssObjectName, LocalFile, OssPartSize, oss.Routines(Routines), oss.Checkpoint(true, ""))
		if err != nil {
			fmt.Println("上传失败，Error:", err)
			os.Exit(-1)
		}


		log.Printf("上传完成, 上传耗时：%v",time.Now().Sub(startTime).Seconds())
	}

}
