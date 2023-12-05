# Clean Code principles

## Names

This has been mentioned before, but for completeness' sake repeated here.

:::{important}
Java is **_CamelCase_** not **_snake_case_**.  
If a name needs a comment, it is not a good name.  
Names should reveal intention of its use and nature. They specify what is measured and the unit in which this happens.
:::

- **variables** are nouns, written lowercase-first. An exception is the `static final` constant, which is written `ALL_CAPS`.
- **methods** are verbs or verb-like names, lowercase-first.
- **classes** are nouns, uppercase-first.

Names _can_ be long. This is no problem with modern IDEs and tools like AI coding assistants.
Of course, it is still a good idea to have a loop counter named `i` because this is convention. Naming it
Here are a few bad examples each followed by a better alternative (without access modifiers): 

```java
//VARIABLES
int qs; //BAD - abbreviation
int quality_score; //BAD - snake_case
int QualityScore; //BAD - first character uppercase

int qualityScore; // GOOD

double dist; //BAD abbreviation
double distance; //BAD - no units

double distanceInMeters; //GOOD

//METHODS
double eucldistnc(){Point x, Point y} //BAD - abbreviation
double distance(Point x, Point y){} //BAD - not informative enough although the next would be OK
double distance(Point x, Point y, DistanceMetric metric){} // GOOD because the metric is a function parameter
double euclideanDistance(Point x, Point y){} //BAD - noun (although this usage is not that bad)
double getEuclideanDistance(Point x, Point y){} //GOOD - noun (although this usage is not that bad)

void process(){} //BAD not informative
void processGffElement(){} //GOOD

//CLASSES
FileParse{} //BAD - not a noun, not informative
Parser{} //BAD - not informative
gffFileParser{} //BAD not uppercase-first
GffFileParser{} //GOOD
```


Here is a special case, a variable holding a complex data structure:

```java
//HashMap that stores employees by ID and employee properties as HashMap as well
//BAD - not coded to interface (Map) and unintelligible
HashMap<String, HashMap<String, String>> employees;

//still BAD - unintelligible
Map<String, Map<String, String>> employees; 

//We have records now!
record Employee(String id, String name, String role){}
//MUCH better
Map<String, Employee> employees;

//maybe even better?
class EmployeeCollection{
    Map<String, Employee> employees;
    //class code to manage employees
}
```

:::{tip}
Refactor complex data structures into data classes (or better - records).
:::

## Methods

### General rules

- Methods have very descriptive names (verb), in `camelCase()` with lowercase-first.
- They do one thing only (SRP!)
- They preferably define no more than 2 or 3 parameters. This keeps testing at an acceptable level of difficulty.

:::{important}
**keep methods testable**
:::


### Keep methods small

- Keep methods short and simple: not too deep nesting of flow control blocks (maximum three) and not too many method parameters. Prefer a small data class (record) as return value over complex data structures.

- Keep a maximum of two indentation levels.

Suppose we have this class, which is a reporting tool on FastQ sequence quality.

```java
package nl.bioinf.refactoring.fastqc;

import java.io.BufferedReader;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.io.File;

public class FastQC {

    public static void main(String[] args) {
        if (args.length != 1) {
            System.out.println("No file provided! \nUsage: java -jar fastqc.jar <fastq file>");
            return;
        }
        File fastqFile = Paths.get(args[0]).toFile();
        if(!fastqFile.exists() || !fastqFile.isFile() || !fastqFile.canRead()) {
            System.out.println("File " + fastqFile + " does not exist");
            return;
        }
        try(BufferedReader reader = Files.newBufferedReader(fastqFile.toPath())) {
            String line;
            String headerLine = "";
            long sequenceCount = 0;
            long failedSequenceCount = 0;
            boolean inQualityLine = false;
            while((line = reader.readLine()) != null) {
                // header lines start with @
                if (line.startsWith("@")) {
                    //System.out.println("Header line: " + line);
                    headerLine = line;
                    sequenceCount++;
                    continue;
                }
                // quality lines start with +
                if (line.startsWith("+")) {
                    //System.out.println("Next is quality line: " + line);
                    inQualityLine = true;
                    continue;
                }

                if (inQualityLine) {
                    //System.out.println("Quality line: " + line);

                    //read the quality line and determine the average quality score
                    //and number of low-quality bases
                    //if the average quality score is below 30, or the number of low-quality
                    //bases higher than 10, mark as failed
                    //score = ascii - 33
                    int lowCount = 0;
                    int summedScore = 0;
                    for (int i = 0; i < line.length(); i++) {
                        char qualityChar = line.charAt(i);
                        int qualityScore = qualityChar - 33;
                        summedScore += qualityScore;
                        if (qualityScore < 20) {
                            lowCount++;
                        }
                    }
                    double averageQuality = (double) summedScore / line.length();
                    //System.out.println("Average Quality: " + +averageQuality + ", low count: " + lowCount);
                    if (averageQuality < 30 || lowCount > 10) {
                        failedSequenceCount++;
                        //System.out.println("Warning: " + headerLine + " has low quality");
                    }
                    inQualityLine = false;
                }
            }
            //report results
            double fraction = (double) failedSequenceCount / sequenceCount * 100;
            System.out.println("Done reading file " + fastqFile);
            System.out.print("Number of failed sequences: " + failedSequenceCount
                    + " out of " + sequenceCount + " sequences ( " +
                    fraction + "%)");
        } catch (Exception e) {
            System.out.println("Error while reading file " + fastqFile);
            e.printStackTrace();
        }
    }
}

```
It has several major flaws:  
- everything is in static `main()`, effectively making this a script instead of reusable code
- its main() has a maximum of 6 indentation levels
- it is not testable
- it is weakly readable code and therefore needs lots of comments
- there are many things happening in th main() method that could be extracted and reused.
- because it has no tests, it needs a lot of print statements.
- (there are probably more)

Let's refactor.  

You can start either from the highest indentation level and work my way up, or the other way around.

I choose the first, because this makes it possible to create JUnit tests along with it.

Here is a snippet with a test that can be extracted into a tiny but highly testable method:

```java
if (averageQuality <= 30 || lowCount >= 10) {
    failedSequenceCount++;
    //System.out.println("Warning: " + headerLine + " has low quality");
}
```

While we're at it, let's take out these within-code quality definitions!
First define some static class variables:

```java
private static final int AVERAGE_QUALITY_CUTOFF = 30;
private static final int LOW_QUALITY_BASES_CUTOFF = 10;
```

and next use these variables:

```java
private static boolean isFailedSequence(double averageQuality, int lowCount) {
    return averageQuality <= AVERAGE_QUALITY_CUTOFF || lowCount >= LOW_QUALITY_BASES_CUTOFF;
}
```

Next, create a test method:

```java
package nl.bioinf.refactoring.fastqc;

import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class FastQCTest {
    @Test
    void isFailedSequence_true() {
        assertTrue(FastQC.isFailedSequence(20, 0));
        assertTrue(FastQC.isFailedSequence(29, 0));
        assertTrue(FastQC.isFailedSequence(35, 11));
        assertTrue(FastQC.isFailedSequence(30, 10));
    }

    @Test
    void isFailedSequence_false() {
        assertFalse(FastQC.isFailedSequence(31, 0));
        assertFalse(FastQC.isFailedSequence(40, 9));
    }
}
```

Finally, here its usage:

```java
if (isFailedSequence(averageQuality, lowCount)) {
    failedSequenceCount++;
}
```

Another tiny, but useful extraction is this (again also using a constant):

```java
static int getQualityScore(char qualityChar) {
    return qualityChar - ILLUMINA_ENCODING;
}
```

And this one 

```java
static boolean isLowQualityBase(char qualityChar) {
    return qualityChar < LOW_QUALITY_SCORE;
}
```

Create a custom small statistics record as inner class:

```java
private record SequenceQuality(int lowCount, int summedScore) { }
```

and use it in a method determining these statistics:

```java
public static SequenceQuality getSequenceQuality(String line) {
    int lowCount = 0;
    int summedScore = 0;
    for (int i = 0; i < line.length(); i++) {
        char qualityChar = line.charAt(i);
        int qualityScore = getQualityScore(qualityChar);
        summedScore += qualityScore;
        if (isLowQualityBase(qualityChar)) {
            lowCount++;
        }
    }
    return new SequenceQuality(lowCount, summedScore);
}
```

Similarly, we can wrap the FastQ file statistics in 

```java
public record FastQfileQuality(long sequenceCount, long failedSequenceCount) {
    public double fractionFailed() {
        return (double) failedSequenceCount / sequenceCount * 100;
    }
}
```

But now it is high time to get out of the `main()` and static environment and get rid of all these `static` keywords.
Here is the end result (maybe it would have been better to separate into multiple source files as well):

```java
package nl.bioinf.refactoring.fastqc;

import java.io.BufferedReader;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.io.File;

public class FastQC {
    //static class variables
    private static final int AVERAGE_QUALITY_CUTOFF = 30;
    private static final int LOW_QUALITY_BASES_CUTOFF = 10;
    private static final int ILLUMINA_ENCODING = 33;
    private static final int LOW_QUALITY_SCORE = 20;

    public static void main(String[] args) {
        if (args.length != 1) {
            System.out.println("No file provided! \nUsage: java -jar fastqc.jar <fastq file>");
            return;
        }
        File fastqFile = Paths.get(args[0]).toFile();
        if (! fileIsOk(fastqFile)) {
            System.out.println("File " + fastqFile + " does not exis or is not readable");
            return;
        }

        FastQC fastQC = new FastQC();
        FastQfileQuality qualityReport = fastQC.readFile(fastqFile);
        System.out.println("Done reading file " + fastqFile);
        System.out.print("Number of failed sequences: " + qualityReport.failedSequenceCount()
                + " out of " + qualityReport.sequenceCount + " sequences (" +
                (qualityReport.fractionFailed()*100) + "%)");
    }

    private record SequenceQuality(int lowCount, int summedScore) { }
    public record FastQfileQuality(long sequenceCount, long failedSequenceCount) {
        public double fractionFailed() {
            return (double) failedSequenceCount / sequenceCount * 100;
        }
    }

    private  boolean fileIsOk(File fastqFile) {
        if(!fastqFile.exists() || !fastqFile.isFile() || !fastqFile.canRead()) {
            System.out.println("File " + fastqFile + " does not exist");
            return false;
        }
        return true;
    }

    private FastQfileQuality readFile(File fastqFile) {
        try(BufferedReader reader = Files.newBufferedReader(fastqFile.toPath())) {
            String line;
            long sequenceCount = 0;
            long failedSequenceCount = 0;
            boolean inQualityLine = false;
            while((line = reader.readLine()) != null) {
                if (isHeaderLine(line)) {
                    sequenceCount++;
                    continue;
                }
                if (isQualityLine(line)) {
                    inQualityLine = true;
                    continue;
                }

                if (inQualityLine) {
                    SequenceQuality sq = getSequenceQuality(line);
                    double averageQuality = (double) sq.summedScore() / line.length();
                    if (isFailedSequence(averageQuality, sq.lowCount())) {
                        failedSequenceCount++;
                    }
                    inQualityLine = false;
                }
            }
            return new FastQfileQuality(sequenceCount, failedSequenceCount);

        } catch (IOException e) {
            throw new RuntimeException("Error while reading file " + fastqFile, e);
        }
    }


    public SequenceQuality getSequenceQuality(String line) {
        int lowCount = 0;
        int summedScore = 0;
        for (int i = 0; i < line.length(); i++) {
            char qualityChar = line.charAt(i);
            int qualityScore = getQualityScore(qualityChar);
            summedScore += qualityScore;
            if (isLowQualityBase(qualityChar)) {
                lowCount++;
            }
        }
        return new SequenceQuality(lowCount, summedScore);
    }

    boolean isQualityLine(String line) {
        return line.startsWith("+");
    }

    boolean isHeaderLine(String line) {
        return line.startsWith("@");
    }

    boolean isLowQualityBase(char qualityChar) {
        return qualityChar < LOW_QUALITY_SCORE;
    }

    int getQualityScore(char qualityChar) {
        return qualityChar - ILLUMINA_ENCODING;
    }

    boolean isFailedSequence(double averageQuality, int lowCount) {
        return averageQuality <= AVERAGE_QUALITY_CUTOFF || lowCount >= LOW_QUALITY_BASES_CUTOFF;
    }
}

```


### Avoid flags



## Constructors



### Prefer factory methods over complex constructors


## Classes

### Place variables in a clear context
