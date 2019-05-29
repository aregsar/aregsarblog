# Installing Laravel Valet on macOS

[installing laravel valet on macOS](https://aregsar.com/blog/2019/installing-laravel-valet-on-macOS)

## Introduction

In this post I will detail the installation and running of Laravel Valet on macOS using the PHP Composer package manager.

Note: Laravel Valet has been ported to Ubuntu Linux. I will provide links for installing Valet on Linux or Windows Subsystem for Linux on Windows 10.

Laravel Valet is a package that consists of a daemon that runs in the background and serves up Laravel sites through an Nginx proxy server. Valet automatically installs Nginx, if it is not already installed, via the Homebrew package manager for macOS.

Valet also installs Dnsmasq via Homebrew. Dnsmasq acts a local DNS server intercepting and routing URIs with the `.test` domain to our local Nginx server.

The URI's that Valet serves will be of the form `<project-name>.test` where the project name in the URL specifies which Laravel project will be served.

> Note: we can configure a different domain then `.test`. You can check out the documentation for Laravel Valet for further info. `https://laravel.com/docs/5.8/valet`.

## Installing Valet via Composer

To install Valet we can run:












