# Makefile for acct_gather_profile/prometheus plugin

AUTOMAKE_OPTIONS = foreign

PLUGIN_FLAGS = -module -avoid-version --export-dynamic

AM_CPPFLAGS = -I$(top_srcdir) -I$(top_srcdir)/src/common $(LIBCURL_CPPFLAGS)

pkglib_LTLIBRARIES = acct_gather_profile_prometheus.la

acct_gather_profile_prometheus_la_SOURCES = acct_gather_profile_prometheus.c
acct_gather_profile_prometheus_la_LDFLAGS = $(SO_LDFLAGS) $(PLUGIN_FLAGS)
acct_gather_profile_prometheus_la_LIBADD = $(LIBCURL)
