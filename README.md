## Lecture 01 - Introduction
### Map函数和Reduce函数

[mapreduce.cpp](../MIT6.824_study/codes/mapreduce.cpp) 用于统计在一组命令行指定的输入文件中，每一个不同的单词出现频率。

```c++
// User’s map function
void Map(const MapInput &input) {
    const string &text = input.value();
    const int n = text.size();
    for (int i = 0; i < n;) {
        // Skip past leading whitespace
        while ((i < n) && isspace(text[i]))
            i++;
        // Find word end
        int start = i;
        while ((i < n) && !isspace(text[i]))
            i++;
        if (start < i)
            Emit(text.substr(start, i - start), "1");
    }
}
```

*Map*函数使用一个key和一个value作为参数。我们这里说的函数是由普通编程语言编写，例如C++，Java等，所以这里的函数任何人都可以写出来。入参中，key是输入文件的名字，通常会被忽略，因为我们不太关心文件名是什么，value是输入文件的内容。所以，对于一个单词计数器来说，value包含了要统计的文本，我们会将这个文本拆分成单词。之后对于每一个单词，我们都会调用*emit*。*emit*由MapReduce框架提供，并且这里的*emit*属于*Map*函数。*emit*会接收两个参数，其中一个是key，另一个是value。在单词计数器的例子中，*emit*入参的key是单词，value是字符串“1”。这就是一个*Map*函数。在一个单词计数器的MapReduce Job中，*Map*函数实际就可以这么简单。而这个*Map*函数不需要知道任何分布式相关的信息，不需要知道有多台计算机，不需要知道实际会通过网络来移动数据。这里非常直观。

---

```c++
// User’s reduce function
void Reduce(ReduceInput *input) {
    // Iterate over all entries with the same key and add the values
    int64 value = 0;
    while (!input->done()) {
        value += StringToInt(input->value());
        input->NextValue();
    }
    // Emit sum for input->key()
    Emit(IntToString(value));
}
```

*Reduce*函数的入参是某个特定key的所有实例（*Map*输出中的key-value对中，出现了一次特定的key就可以算作一个实例）。所以*Reduce*函数也是使用一个key和一个value作为参数，其中value是一个数组，里面每一个元素是*Map*函数输出的key的一个实例的value。对于单词计数器来说，key就是单词，value就是由字符串“1”组成的数组，所以，我们不需要关心value的内容是什么，我们只需要关心value数组的长度。*Reduce*函数也有一个属于自己的*emit*函数。这里的*emit*函数只会接受一个参数value，这个value会作为*Reduce*函数入参的key的最终输出。所以，对于单词计数器，我们会给emit传入数组的长度。这就是一个最简单的*Reduce*函数。并且*Reduce*也不需要知道任何有关容错或者其他有关分布式相关的信息。