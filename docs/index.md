# Weights & Biases Documentation

## Intro

[Weights & Biases](http://wandb.com) tracks machine learning jobs in real-time, makes them reproducible, and permanently archives your results.

## Quickstart - Existing Project

This explains how to quickly integrate wandb into an existing project.

<br>
After [signing up](https://app.wandb.ai/login), install the Weights & Biases command line tool "wandb":
```console
$ pip install wandb
```

<br>
Initialize Weights & Biases in your project:
```console
$ cd <project_directory>
$ wandb init
```

Follow the prompts to complete the initialization process.

<br>
Then, import and initialize wandb. In your training script:
```python
import wandb
wandb.init()
```

<br>
Finally, launch your job:
```console
wandb run --show <train.py>
```

## Weights & Biases Run API

The simplest way to use Weights & Biases is to call `wandb.init()` from your script, and then launch it with `wandb run <script.py>`. We'll collect and store your job's output, and other metadata. For more advanced usage, like saving produced files, configuration, and statistics, you interact with the `Run` object returned by wandb.init(). Read on!

In the following sections, `run` is the return value of `wandb.init()`:

```python
import wandb
run = wandb.init()
```

### Saving run files

Each time you run your script, by default we create a unique directory that you can store job outputs in. The path is `./wandb/run-<timestamp>-<runid>` and is available to your job as `run.dir`.

Any files you save in this directory during the run will be persisted to Weights & Biases. We recommend that you modify your training script to save generated models and other run artifacts in this directory.

#### Aside: Saving all generated files without modifying your script

You can use `wandb run --dir=. <script.py>` to make wandb sync _all_ files (in the current directory and all descendent directories) generated by your script. This isn't the preferred approach because it won't work well for running parallel instances of your script on a single machine, but it can be useful in some cases.

#### Dry run mode

If you launch your script directly (`python <script.py>` or `./script.py`) instead of using `wandb run <script.py>`, it will be launched in dry run mode, wherein nothing is persisted to wandb. We still create a run directory and other `run` members, so your script will function normally. We recommend using dry run mode while developing your script.

### Saving and loading configuration

During your run, `run.config` is a dict-like that serves two purposes: 1) whatever you put into it will be persisted as the configuration for this run, 2) `wandb run` can optionally read in values from yaml files, and pass them to your script via `run.config`.

#### Saving parameters from argparse / tensorflow / other

If you are using argparse you can easily persist your arguments:

```python
parser = argparse.ArgumentParser()
parser.add_argument('--epochs', type=int, default=10)
args = parser.parse_args()

run.config.update(args)
```

If you're using tensorflow's flags, you can easily persist those:

```python
flags = tf.app.flags
flags.DEFINE_integer('epochs', 10, 'Number of epochs')

run.config.update(flags.FLAGS)
```

You can also set individual keys or attributes on the object.

#### Using yaml files to configure your runs

You can use the wandb config system to pass in configurations from yaml files, rather than via command line parameters. Or you can use both: yaml files to pass in defaults, and command line parameters to override them.

`wandb run` will always look for a file called "config-defaults.yaml" in the root directory of your project. If "config-defaults.yaml" is present it populates the contained values in `run.config` when your run starts. This file is generated when you run `wandb init`, and contains some example variables.

You can pass extra config files with `wandb run --configs=<config_files>` where `<config_files>` is a comma-separated list of paths to yaml files. These will be processed in order, with values from later files overriding values from earlier files.

### Saving metrics

wandb's history and summary allow you to save data about your training run.

#### run.history

history is for storing data that changes over time. For example, if your run proceeds by epochs, you should probably save one data point per epoch, with loss, validation accuracy and other metrics. Just call `run.history.add(<data>)` for each epoch, where `<data>` is a dict of `key: value` pairs.

Weights & Biases shows plots of this data in real-time as your job runs.

#### run.summary

This is a dict-like object used to summarize the results of your run. We recommend storing information that you'd use to compare this run to other runs. For example, you could store the highest accuracy that your run achieved.

Weights & Biases shows these values in the runs page for your project.

#### Automatically generating history & summary with Keras

You can use `wandb.keras.WandbCallback()` to automatically generate summary and history data. It will also save the best model, like `keras.ModelCheckpoint`. Just instantiate it and pass it in to Keras' `model.fit` as a callback.
