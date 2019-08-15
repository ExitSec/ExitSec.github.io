---
layout: post
title: ML.NET Review - MultiClass Classification Batch Issue Extension
category: Code
---

# Introduction

There is an extremely large amount of hype around the field of artificial intelligence and its subcategories. Of interest to us is the Machine Learning (ML) component of AI. While there are many libraries out in the wild, such as, [PyTorch](https://pytorch.org/) and [Scikit-Learn](https://scikit-learn.org/). Microsoft has begun to develop their own library structure that leverage's the .NET framework. Therefore, for this blog post, I think it would be good to explore the sophistication of ML.NET through a review of their MultiClass Classification sample, but see how easy it is to adapt to allow for Batch Predictions.

What is discussed in this post:

* Introduction to ML.NET
* C# MultiClass Classification Review
* A C# Adaption of MultiClass Classification within ML.NET

Let's begin...


# ML.NET and groundings

With the growth of the .NET framework and the native integration of Microsoft products, many developers have migrated from other languages such as Java and C++ to Microsoft's C# language. While C# has been typically a driver for the pure Microsoft market, the drive to grow beyond this has been the driver for .NET Core and the cross-platform integration. As the influx of C# developers continues, the need to broaden the language further was required.

Typically, Python has been the main language in which developers use for data analysis and ML. Therefore, Python has established its dominance within this field and has been an extremely difficult to contend with due to the low-level of entry required for the language and frameworks.

With the growth of the C# language and open source initiative, the birth of ML.NET on [May 7th 2018](https://devblogs.microsoft.com/dotnet/introducing-ml-net-cross-platform-proven-and-open-source-machine-learning-framework/). However, ML.NET has not been a competitor to earlier machine learning frameworks such as Tensorflow, but a facilitator to incorporate and extend such frameworks.

The current development stage which has been stable and officially released as a major revision is ML.NET 1.0 on [May 6th 2019](https://devblogs.microsoft.com/dotnet/announcing-ml-net-1-0/). ML.NET has been the ML component for many of the Windows products e.g., Windows Defender, Microsoft Office, Azure Machine Learning and PowerBI. With the major release of ML.NET, the level of entry is significantly low with the ability to auto generate models through the [Model Builder](https://marketplace.visualstudio.com/items?itemName=MLNET.07), along with many different [samples](https://github.com/dotnet/machinelearning-samples) across a breadth of algorithms.

Working with a framework can be extremely hard to begin with an can take a large portion of time to get proficient with it. Many organisations seem to pride themselves on their documentation, however, Microsoft is one that I can say does deliver on good quality documentation. While the documentation is of good quality, the framework is fairly new,therefore has is limited in comparison to that of a mature framework. For anyone looking for the ML.NET documentation, you can find it [here](https://docs.microsoft.com/en-us/dotnet/machine-learning/).

With all of this going for the ML.NET framework, I have found it a great way of carrying out experimental analysis for my PhD, along with a better understanding of the ML field. Let us have a look at an example through the proposed Microsoft [GitHub Issues](https://docs.microsoft.com/en-us/dotnet/machine-learning/tutorials/github-issue-classification) example using a MultiClass Classification algorithm.

# Working with ML.NET
Before jumping into a project from scratch using a completely new framework, I thought it best to get a good understanding of the work flow and understanding on how to adapt and evolve my approach to fit the bill of a new project, while still having the safety of an example to fall back on. For my future blog post, I am to use a MultiClass Classification algorithm to make predictions on Intrusion Detection data. Luckily, Microsoft has provided the [MultiClass Classification](https://docs.microsoft.com/en-us/dotnet/machine-learning/tutorials/github-issue-classification) example to work with, in which I have followed but adapt to achieve answers to the questions:

* How to create manual multi inputs and file-based inputs
* How to make predictions based on multi inputs and file-based inputs

While many of the examples work on the basis of a singular input for prediction, which is based on the premise of data is hard to acquire. There are scenarios in which we have training, testing, and unlabelled data such as intrusion detection datasets like KDDCups and Tokyo2006.


## MultiClass Classification Review

As previously discussed, the documentation of the Microsoft libraries tends to be extremely well put together. In doing so, allows for the early adoption of new libraries to be easier and more accessible. As one of the main key driving points of Microsoft's ML.NET is the accessibility to its library. The MultiClass Classification sample presents a working understanding of implementing a full working pipeline with the ability to train, evaluate, save, load, and predict in a seamlessly and efficient manner. While working through the example, interest should be paid to the different metrics and training mechanics that can be implemented. For example, SdcaMaximumEntropy is used as our MultiClassification Training algorithm, as shown below.

*Training*
```cs
// Program.cs
var trainingPipeline = pipeline
    .Append(_mlContext.MulticlassClassification.Trainers.SdcaMaximumEntropy("Label", "Features"))
    .Append(_mlContext.Transforms.Conversion.MapKeyToValue("PredictedLabel"));
```

While the documentation does not explain what the SDCAMaximumEntropy algorithm is, from the best of my knowledge and understanding of the code, the SDCAMaximumEntropy is typically used to make predictions on missing words, in this case an Area. It takes the least informative amount of information which in this case is the provided data, which is then used to make a prediction on missing data in this case area when we supply only partial sets of information such as Title and Description. More information on this algorithm can be found at: [Understanding Deep Learning Generalization by Maximum Entropy](https://arxiv.org/pdf/1711.07758.pdf).

If you would like to see what other algorithms that can be used use the intellisense after the final fullstop of Trainers
```cs
// Program.cs
var trainingPipeline = pipeline
    .Append(_mlContext.MulticlassClassification.Trainers.<algorithms>("Label", "Features"))
```

*Metrics*
```cs
// Program.cs
var testMetrics = _mlContext.MulticlassClassification.Evaluate(_trainedModel.Transform(testDataView));
Console.WriteLine($"*************************************************************************************************************");
Console.WriteLine($"*       Metrics for Multi-class Classification model - Test Data     ");
Console.WriteLine($"*------------------------------------------------------------------------------------------------------------");
Console.WriteLine($"*       MicroAccuracy:    {testMetrics.MicroAccuracy:0.###}");
Console.WriteLine($"*       MacroAccuracy:    {testMetrics.MacroAccuracy:0.###}");
Console.WriteLine($"*       LogLoss:          {testMetrics.LogLoss:#.###}");
Console.WriteLine($"*       LogLossReduction: {testMetrics.LogLossReduction:#.###}");
Console.WriteLine($"*************************************************************************************************************");
```

If you would like to see other metrics that the algorithm can offer, adapt testMetrics as follows
```cs
Console.WriteLine($"*       metric:          {testMetrics.<metric>}");
```

The end section of the example demonstrates the use of a singular prediction to determine the overall implementation that has been presented.

```cs
Console.WriteLine($"=============== Single Prediction - Result: {prediction.Area} ===============");
```

As previously discussed, the justification for a single prediction is that there is not always a plethora of information that could be used for batch predictions. However, Microsoft demonstrate that the use of the Transform function can be used to formulate batch prediction as shown [here](https://docs.microsoft.com/en-us/dotnet/machine-learning/tutorials/sentiment-analysis#predict-comment-sentiment). From my experience this is **NOT** the case and can be removed from the code.

I have provided an example with an extended singular prediction to incorporate multiple entries, along with a file-based batch prediction method. This is through the justification that Intrusion Detection datasets are typically formulated with a test/training dataset along with a unlabelled dataset.

## Batch Prediction - Multiple Manual Entries

To start off, let us look at extending the singular prediction methodology to incorporate a multitude of entries. First, we need to create a [IEnumerable](https://docs.microsoft.com/en-us/dotnet/api/system.collections.ienumerable?view=netframework-4.8) which creates a iterator across a specific type in this case a GitHubIssue.

```cs
IEnumerable<GitHubIssue> issues = new[]
{
    new GitHubIssue
    {
            Title = "Entity Framework crashes",
            Description = "When connecting to the database, EF is crashing"
    },
    new GitHubIssue
    {
        Title = "Github Down",
        Description = "When going to the website, github says it is down"
    }
};
```

Next we will define a _predEngine object to access our batchPredictions.

```cs
var batchPrediction = _predEngine;
```

The prediction engine typically works off a singular entity at a time, therefore the previous single prediction can be passed directly in, as shown below:

```cs
var singleprediction = _predEngine.Predict(singleIssue);
```

However, in the case of batch predictions this cannot be done. Instead we must load the data from our Enumerable into our mlContext pipeline, which can be done through the mlContext.data component of the framework.
```cs
IDataView batchIssues = _mlContext.Data.LoadFromEnumerable(issues);
```

Next we must map our loaded data in the pipeline to the GitHubIssue structure, through creating another enumerable:

```cs
 IEnumerable<GitHubIssue> predictedResults =
     _mlContext.Data.CreateEnumerable<GitHubIssue>(batchIssues, reuseRowObject: false);
```

Finally, to make predictions we must break down the batch of issues into a singular entity as previously discussed, the prediction engine must work on a singular bit of data at a time. To achieve this, we can use a simple foreach structure as follows:
```cs
foreach (GitHubIssue prediction in predictedResults)
{
    Console.WriteLine($"============================== Enumerated-based Batch Predictions ===================================");
    Console.WriteLine($"*-> Title: {prediction.Title} | Prediction: {batchPrediction.Predict(prediction)}");
}
```
Using our prediction engine via the batchPrediction handler, we can now pass a single prediction from our batch to be analysed at any given time. Thus, satisfying the overall limitation of single bits of data.

The full example is as follows:
```cs
ITransformer loadedModel = _mlContext.Model.Load(_modelPath, out var modelInputSchema);
_predEngine = _mlContext.Model.CreatePredictionEngine<GitHubIssue, IssuePrediction>(loadedModel);

/*
* IEnumerable-based Batch Predictions
* The following block of code, demonstrates the creation of manual entries and storing
 * them into area. Each of the indexes within the array is then predicted via the _predEngine.Predict function
*/

IEnumerable<GitHubIssue> issues = new[]
{
    new GitHubIssue
    {
        Title = "Entity Framework crashes",
        Description = "When connecting to the database, EF is crashing"
    },
    new GitHubIssue
    {
        Title = "Github Down",
        Description = "When going to the website, github says it is down"
    }
};

var batchPrediction = _predEngine;


IDataView batchIssues = _mlContext.Data.LoadFromEnumerable(issues);


IEnumerable<GitHubIssue> predictedResults =
    _mlContext.Data.CreateEnumerable<GitHubIssue>(batchIssues, reuseRowObject: false);

foreach (GitHubIssue prediction in predictedResults)
{
    Console.WriteLine($"============================== Enumerated-based Batch Predictions ===================================");
    Console.WriteLine($"*-> Title: {prediction.Title} | Prediction: {batchPrediction.Predict(prediction)}");
}

```

There we have it we can access the 'Area' prediction directly through this method and outputs to what we see within the singular prediction, showing that this does indeed work. However, the question remains, what if I want to pass in files, from an automated source?

## Batch Prediction - File-based Data
For anyone wanting to try this out I have provided a sample of unlabelled test data, which can be found here: [Unlabelled Test Data](/misc_files/ml_githubissues/issues_unlabelled.tsv) (*it is important that it follows the same structure as our training and test data, unless you would like to make a new data structure*), once you have downloaded the contents you can place it alongside the other .tsv files in the Data directory.

The first component required is to provide our unlabelled file of batch predictions. Within the global scope of Class Program, place the following two lines of code.
```cs
// Program.cs Global Variable
private static string _myTestDataPath => Path.Combine(_appPath, "..", "..", "..", "Data", "issues_unlabelled.tsv");
private static IDataView _myTestDataView;
```
The first line defines the path to find out unlabelled test data, and the second creates a IDataView array to be used to store our contents. To load our contents into the mlContext pipeline we must load from a textfile, instead of an enumerable. This can be done with the LoadFromTextFile function within the mlContext.Data component of the framework.
```cs
_myTestDataView = _mlContext.Data.LoadFromTextFile<GitHubIssue>(_myTestDataPath, hasHeader: true);
```

Next, we create a new enumerable for our predicted results to be mapped to our defined data structure. This is almost the same as the batch of multiple manual entries we discussed previously.

```cs
IEnumerable<GitHubIssue> filePredictedResults =
    _mlContext.Data.CreateEnumerable<GitHubIssue>(_myTestDataView, reuseRowObject: false);
```

Finally, we need to access a single prediction at a time, we again require a foreach.
```cs
foreach (GitHubIssue prediction in filePredictedResults)
{
    Console.WriteLine($"================================= File-based Batch Predictions ======================================");
    Console.WriteLine($"*-> Title: {prediction.Title} | Prediction: {batchPrediction.Predict(prediction).Area}");
}

```
However, one key difference here is that we need to access the '.Area' property of the prediction, otherwise we are given the type of data structure the prediction is. Finally, we should have the same results as our manual batch entries and singular predictions.


The full example is as follows:
```cs
// Program.cs -> Global Variable
private static string _myTestDataPath => Path.Combine(_appPath, "..", "..", "..", "Data", "issues_unlabelled.tsv");
private static IDataView _myTestDataView;


// Program.cs -> PredictIssue

ITransformer loadedModel = _mlContext.Model.Load(_modelPath, out var modelInputSchema);
_predEngine = _mlContext.Model.CreatePredictionEngine<GitHubIssue, IssuePrediction>(loadedModel);

/*
* File-based Batch Predictions
* The following block of code, demonstrates the ingestion of a file (myTestData.tsv) which has no
* Area, to then predicted via the _predEngine.Predict function.
*/

_myTestDataView = _mlContext.Data.LoadFromTextFile<GitHubIssue>(_myTestDataPath, hasHeader: true);

IEnumerable<GitHubIssue> filePredictedResults =
     _mlContext.Data.CreateEnumerable<GitHubIssue>(_myTestDataView, reuseRowObject: false);

foreach (GitHubIssue prediction in filePredictedResults)
{
    Console.WriteLine($"================================= File-based Batch Predictions ======================================");
    Console.WriteLine($"*-> Title: {prediction.Title} | Prediction: {batchPrediction.Predict(prediction).Area}");
}

```

# Conclusion
Working with the ML.NET framework is extremely delightful. The simplicity and elegance of the framework demonstrates a lot of thought and careful design. Further development within this framework will allow for the ease of accessibility to newcomers of the field, but also allow for the adoption of it within many future products. By seeing the adoption within Microsoft, provides a high level of confidence to me that this framework is something that should be definitely adopted sooner rather than later! The only issue I have at the minute was a little lack of further work to the documentation, however, this is something I can see being changed as the framework matures.

The last three days of working with the framework has been extremely enjoyable and has allowed the logical representation of the Machine Learning process to be much clearer than another implementation. I strongly recommend the testing and utilisation of the ML.NET framework for anyone that is looking for a fun few days.
