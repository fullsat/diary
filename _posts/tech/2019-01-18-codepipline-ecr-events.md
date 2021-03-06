---
title:  "CodePipelineのECRソースはCloudWatchEventsで実現されている"
date:   2019-01-18 18:00:00 +0900
category: tech
tags: aws terraform
layout: post
---

ということに気づかず、terraformで作ったCodePipelineがなぜか起動しないという罠がありました。

## 現象

* aws_codepipelineリソースをterraformで作成した
* そのリソース内にECRをソースにしたステージを定義した
* terraform applyで作成し、コンソールからリリースして成功した
* ただコンソールからは実行できるが、ECRにイメージをpushした時には動かなかった

## 原因

CloudWatchEventsのリソースを追加する必要がありました。

CodePipelineを動かすのは謎の技術ではなく、
CloudWatchEventsの力で動いていました。

全く意識してませんでしたが、CodePipelineをコンソール上から作るときに、
裏側で自動的にCloudWatchEventsが作成されています。

したがって、terraformで作成する際は以下のようにイベントを自分で追加する必要があります。

```terraform:codepipline.tf
resource "aws_cloudwatch_event_rule" "pushed" {
  name        = "pushed"
  description = "CodePipelineを起動するためのイベント"

  event_pattern = <<PATTERN
{
  "source": [
    "aws.ecr"
  ],
  "detail": {
    "eventName": [
      "PutImage"
    ],
    "requestParameters": {
      "repositoryName": [
        "${aws_ecr_repository.tmp-pipeline.name}"
      ],
      "imageTag": [
        "latest"
      ]
    }
  }
}
PATTERN
}

resource "aws_cloudwatch_event_target" "pushed" {
  target_id = "pushed"
  rule      = "${aws_cloudwatch_event_rule.pushed.name}"
  arn       = "${aws_codepipeline.deploy.arn}"
  role_arn  = "${aws_iam_role.execute.arn}"
}


resource "aws_iam_role" "execution" {
  name               = "execute"
  path               = "/"
  description        = "Execute codepipeline project"
  assume_role_policy = <<POLICY
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "",
      "Effect": "Allow",
      "Principal": {
        "Service": [
          "events.amazonaws.com"
        ]
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
POLICY
}

resource "aws_iam_role_policy" "execute" {
  name        = "execute"
  role        = "${aws_iam_role.execute.name}"
  policy      = <<POLICY
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": [
        "codepipeline:StartPipelineExecution"
      ],
      "Resource": [ "${aws_codepipeline.deploy.arn}" ],
      "Effect": "Allow"
    }
  ]
}
POLICY
}

```

# まとめ

こんなん気づくかー、と叫びたくなりました。

ここから教訓を得るなら、何事もどのようにして動いているのか、を追いかけていく必要があるということですね。

