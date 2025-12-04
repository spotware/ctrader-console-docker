# cTrader Console Docker Image

## About this Image

This official container images for cTrader Console on Linux for Docker Engine.

## How to use this Image

You can run cTrader console by pulling this image and then running it.

Pull:

```
docker pull ghcr.io/spotware/ctrader-console:5.4
```

Run cBot:

```
docker run -d -it --name ctrader.console.run.mybot --mount type=bind,src=/cAlgo/Robots,dst=/mnt/Robots -e CTID='mycid' -e PWD-FILE='/mnt/Robots/ctrader-cli.pwd' -e ACCOUNT='9102302' -e SYMBOL='EURUSD' -e PERIOD='H1' ghcr.io/spotware/ctrader-console:latest run "/mnt/Robots/My bot.algo" --environment-variables
```

Backtest cBot:

```
docker run -d -it --name ctrader.console.run.mybot --mount type=bind,src=/cAlgo/Robots,dst=/mnt/Robots -e CTID='mycid' -e PWD-FILE='/mnt/Robots/ctrader-cli.pwd' -e ACCOUNT='9102302' -e SYMBOL='EURUSD' -e PERIOD='H1' -e START="01/01/2025" -e END="01/02/2025" -e DATA-MODE='m1' -e BALANCE='10000' -e COMMISSION='15' -e SPREAD='1' -e REPORT='/mnt/Robots/report.html' -e REPORT-JSON='/mnt/Robots/report.json' ghcr.io/spotware/ctrader-console:latest backtest "/mnt/Robots/Sample Breakout cBot.algo" --environment-variables --full-access
```

We use Docker mount to let the container have access to a directory so it will be able to run the cBot algo file.

In above command we use:

* `ctrader.console.run.mybot`: Container name, you can use any name
* `/cAlgo/Robots`: Path in host that will be mounted to container
* `/mnt/Robots`: Container mount path
* `CTID='mycid'`: Your cID username
* `/mnt/Robots/ctrader-cli.pwd`: Your cID password file path according to mount path
* `ACCOUNT='9102302'`: Your trading account number
* `SYMBOL='EURUSD'`: Symbol that cBot will run on
* `PERIOD='H1'`: Timeframe that cBot will run on
* `ghcr.io/spotware/ctrader-console:5.4`: Container image name that you pulled, change the tag if you pulled some specific version
* `/mnt/Robots/My bot.algo`: Your cBot algo file path according to mount path
* `environment-variables`: We use this parameter to let cTrader console know it should use environment variables for getting its parameters

In above command we used Docker mount to let container have access to our cBot algo file and cID password file, instead you can also use `-volume` or build another docker file based on console image and copy your algo files and set environment variables directly inside your docker file.

For more regarding Docker mount and volume please check Docker documentation.

You can use all of the cTrader console commands like backtest, for more regarding available features in cTrader Console please check the documentation.

## Tags

We use latest for latest available version, to use specific version you can pull it by using the version as tag, ex:

```
docker pull ghcr.io/spotware/ctrader-console:5.4
```

## Useful Links

[cTrader CLI Documentation](https://help.ctrader.com/ctrader-algo/ctrader-cli/)
