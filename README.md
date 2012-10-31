cascading.pattern
=================

_Pattern_ sub-project for Cascading.org which uses flows as containers
for machine learning models, importing
[PMML](http://en.wikipedia.org/wiki/Predictive_Model_Markup_Language)
model descriptions from _R_, _SAS_, _Weka_, _RapidMiner_, _SQL
Server_, etc.

Currently supported algorithms for PMML include:

 * [Random Forest](http://en.wikipedia.org/wiki/Random_forest)


Build Instructions
------------------

To build _Pattern_ and then run its unit tests:

    gradle --info --stacktrace clean test

The following scripts generate a baseline for the _Random Forest_
algorithm. This baseline includes a reference data set (simulated
ecommerce orders) plus a predictive model in PMML:

    ./src/py/rf_sample.py 200 > data/orders.tsv
    R --vanilla --slave < src/r/rf_model.R > model.log

To build _Pattern_ and then run this baseline test:

    gradle clean jar
    rm -rf output
    hadoop jar build/libs/pattern.jar data/sample.xml data/sample.tsv output/classify output/measure output/trap

For each tuple in the data, a _stream assertion_ tests whether the
`predicted` field matches the `score` field generated by the
model. Tuples which fail that assertion get trapped into
`output/trap/part*` for inspection.

Also, the _confusion matrix_ shown in `output/measure/part*` should
match the one logged in `model.log` from baseline generated in _R_.


Usage
-----

Alternatively, if you just want to re-use this assembly for your own
Cascading app, you can remove `verifyPipe` and `measurePipe` from the
sample code and copy it into your app.

The following snippet shows how to generate a PMML file `sample.xml`
from a Random Forest model trained in R:

    f <- as.formula("as.factor(label) ~ .")
    fit <- randomForest(f, data_train, ntree=50)
    saveXML(pmml(fit), file="sample.xml")


Then to use the PMML file in your Java code, referenced as a command
line argument called `pmmlPath`:

    // define a "Classifier" model from PMML to evaluate the orders
    Classifier model = ClassifierFactory.getClassifier( pmmlPath );
    Pipe classifyPipe = new Each( new Pipe( "classify" ), model.getFields(), new ClassifierFunction( new Fields( "score" ), model ), Fields.ALL );

Now when you run that Cascading app, provide a reference to
`sample.xml` for the `pmmlPath` argument.


PMML Resources
--------------

 * [Data Mining Group](http://www.dmg.org/) XML standards and supported vendors
 * [PMML In Action](http://www.amazon.com/dp/1470003244) book 
 * [PMML validator](http://www.zementis.com/pmml_tools.htm)
