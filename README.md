
import http from "k6/http";
import { check } from "k6";

const rate = Number(__ENV.RATE || 100);

export const options = {
  scenarios: {
    api_load: {
      executor: "constant-arrival-rate",
      rate,
      timeUnit: "1s",
      duration: "30m",
      preAllocatedVUs: 100,
      maxVUs: 1000,
    },
  },
  thresholds: {
    http_req_failed: ["rate<0.005"],
    http_req_duration: ["p(95)<500"],
    dropped_iterations: ["count==0"],
  },
};

export default function () {
  const response = http.get(
    `${__ENV.BASE_URL}/api/v1/items?limit=20`,
    {
      headers: {
        Authorization: `Bearer ${__ENV.TEST_TOKEN}`,
      },
      tags: { endpoint: "items-list" },
    }
  );

  check(response, {
    "status 200": (r) => r.status === 200,
  });
}
