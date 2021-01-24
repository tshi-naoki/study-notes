修改自：https://www.cnblogs.com/gaizai/archive/2010/05/27/1743485.html

# UriUtils

```c#
public class UriUtils
{
    public static string GetUriQuery(string webLink, string link)
    {
        var uri = new Uri(webLink);
        var queryString = uri.Query;
        var col = GetQueryString(queryString);
        return col[link];
    }

    /// <summary>
    /// 将查询字符串解析转换为名值集合
    /// </summary>
    /// <param name="queryString"></param>
    /// <returns></returns>
    public static NameValueCollection GetQueryString(string queryString)
    {
        return GetQueryString(queryString, null, true);
    }

    /// <summary>
    /// 将查询字符串解析转换为名值集合
    /// </summary>
    /// <param name="queryString"></param>
    /// <param name="encoding"></param>
    /// <param name="isEncoded"></param>
    /// <returns></returns>
    public static NameValueCollection GetQueryString(string queryString, Encoding encoding, bool isEncoded)
    {
        queryString = queryString.Replace("?", "");
        var result = new NameValueCollection(StringComparer.OrdinalIgnoreCase);
        if (!string.IsNullOrEmpty(queryString))
        {
            int count = queryString.Length;
            for (int i = 0; i < count; i++)
            {
                int startIndex = i;
                int index = -1;
                while (i < count)
                {
                    char item = queryString[i];
                    if (item == '=')
                    {
                        if (index < 0)
                        {
                            index = i;
                        }
                    }
                    else if (item == '&')
                    {
                        break;
                    }
                    i++;
                }
                var key = string.Empty;
                var value = string.Empty;
                if (index >= 0)
                {
                    key = queryString.Substring(startIndex, index - startIndex);
                    value = queryString.Substring(index + 1, (i - index) - 1);
                }
                else
                {
                    key = queryString.Substring(startIndex, i - startIndex);
                }
                if (isEncoded)
                {
                    result[MyUrlDeCode(key, encoding)] = MyUrlDeCode(value, encoding);
                }
                else
                {
                    result[key] = value;
                }
                if ((i == (count - 1)) && (queryString[i] == '&'))
                {
                    result[key] = string.Empty;
                }
            }
        }
        return result;
    }

    /// <summary>
    /// 解码URL
    /// </summary>
    /// <param name="encoding">null为自动选择编码</param>
    /// <param name="str"></param>
    /// <returns></returns>
    public static string MyUrlDeCode(string str, Encoding encoding)
    {
        if (encoding == null)
        {
            var utf8 = Encoding.UTF8;
            //首先用utf-8进行解码                     
            var code = HttpUtility.UrlDecode(str.ToUpper(), utf8);
            //将已经解码的字符再次进行编码.
            string encode = HttpUtility.UrlEncode(code, utf8).ToUpper();
            if (str == encode)
                encoding = Encoding.UTF8;
            else
                encoding = Encoding.GetEncoding("gb2312");
        }

        return HttpUtility.UrlDecode(str, encoding);
    }
}
```

# 调用

```c#
private string GetNewWebLink(string webLink)
{
    var link = UriUtils.GetUriQuery(webLink, "link");
    Console.WriteLine(link);
    return url;
}
```