cascading.pattern
=================

_Pattern_ sub-project for http://Cascading.org/ which uses flows as
containers for machine learning models, importing
[PMML](http://en.wikipedia.org/wiki/Predictive_Model_Markup_Language)
model descriptions from _R_, _SAS_, _Weka_, _RapidMiner_, _KNIME_,
_SQL Server_, etc.

Current support for PMML includes:

 * [Random Forest](http://en.wikipedia.org/wiki/Random_forest) in [PMML 4.x](http://www.dmg.org/v4-0-1/GeneralStructure.html) exported from [R/Rattle](http://cran.r-project.org/web/packages/rattle/index.html)

This is intended to complement other ML libraries atop Cascading, such as the excellent
[Scalding Matrix API](https://github.com/twitter/scalding/wiki/Matrix-API-Reference).


Project Status
--------------

One struggles with the notion of calling this an _alpha_ release.
More like a _petroglyph_ release; very very early.

We wanted to get the code "out there" early, to initiate a dialog and
collect feedback about using PMML and Cascading together.


Build Instructions
------------------

To build _Pattern_ and then run its unit tests:

    gradle --info --stacktrace clean test

The following scripts generate a baseline (model+data) for the _Random
Forest_ algorithm. This baseline includes a reference data set
(simulated ecommerce orders) plus a predictive model in PMML:

    ./src/py/pmml_orders.py 200 > data/orders.tsv
    R --vanilla --slave < src/r/pmml_models.R > model.log

To build _Pattern_ and then run the baseline regression test:

    gradle clean jar
    rm -rf output
    hadoop jar build/libs/pattern.jar data/sample.rf.xml data/sample.tsv output/classify output/trap --measure output/measure --assert

For each tuple in the data, a _stream assertion_ tests whether the
`predicted` field matches the `score` field generated by the
model. Tuples which fail that assertion get trapped into
`output/trap/part*` for inspection.

Also, the _confusion matrix_ shown in `output/measure/part*` should
match the one logged in `model.log` from baseline generated in _R_.


Usage
-----

Alternatively, if you want to re-use this assembly for your own
Cascading app, remove the parts related to `verifyPipe` and
`measurePipe` from the sample code.

The following snippet in R shows how to train a Random Forest model,
then generate PMML as a file called `sample.rf.xml`:

    f <- as.formula("as.factor(label) ~ .")
    fit <- randomForest(f, data_train, ntree=50)
    saveXML(pmml(fit), file="sample.rf.xml")


Then to use the PMML file in your Java code, referenced as a command
line argument called `pmmlPath`:

    // define a "Classifier" model from PMML to evaluate the orders
    Classifier model = ClassifierFactory.getClassifier( pmmlPath );
    ClassifierFunction classFunc = new ClassifierFunction( new Fields( "score" ), model );
    Pipe classifyPipe = new Each( new Pipe( "classify" ), model.getFields(), classFunc, Fields.ALL );

Now when you run that Cascading app, provide a reference to
`sample.rf.xml` for the `pmmlPath` argument.

An architectural diagram for common use case patterns is shown in
`docs/pattern.graffle` which is an OmniGraffle document.

Examples
--------

Check in the `src/r/rattle_pmml.R` source code for examples of
predictive models created in R, then exported using _Rattle_.
These examples use the popular _Iris_ data set.

 * random forest (rf)
 * recursive partition classification tree (rpart)
 * single hidden-layer neural network (nnet)
 * multinomial model (multinom)
 * linear regression (lm)
 * logistic regression (glm)
 * support vector machine (ksvm)
 * k-means clustering (kmeans)
 * hierarchical clustering (hclust)

PMML Resources
--------------

 * [Data Mining Group](http://www.dmg.org/) XML standards and supported vendors
 * [PMML In Action](http://www.amazon.com/dp/1470003244) book 
 * [PMML validator](http://www.zementis.com/pmml_tools.htm)
